# kind 로 구성한 클러스터에 metallb 로드밸런서 컨트롤러 설치

1. 클러스터에 metallb 설치

        kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/refs/heads/main/config/manifests/metallb-native-prometheus.yaml   


        (base) root@djlee:/home/djlee/projects/k8s# kubectl get all -n metallb-system
        NAME                              READY   STATUS              RESTARTS   AGE
        pod/controller-5fc458bdcf-cm282   2/2     Running             0          35s
        pod/speaker-4f287                 0/2     ContainerCreating   0          35s
        pod/speaker-gc9j7                 0/2     ContainerCreating   0          35s
        pod/speaker-jgjgc                 0/2     ContainerCreating   0          35s
        pod/speaker-zjrqz                 0/2     ContainerCreating   0          35s

        NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
        service/controller-monitor-service   ClusterIP   None            <none>        9120/TCP   35s
        service/metallb-webhook-service      ClusterIP   10.96.171.251   <none>        443/TCP    35s
        service/speaker-monitor-service      ClusterIP   None            <none>        9120/TCP   35s

        NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
        daemonset.apps/speaker   4         4         0       4            0           kubernetes.io/os=linux   35s

        NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/controller   1/1     1            1           35s

        NAME                                    DESIRED   CURRENT   READY   AGE
        replicaset.apps/controller-5fc458bdcf   1         1         1       35s

2. metallb 가 제공할 IP address Pool 과 각 Service 들에게 어떤 IP를 쓸 수 있는지 광고 (Advertise) 할 방법을 yaml 파일로 작성하여 배포

    #### metallb-config.yaml

        apiVersion: metallb.io/v1beta1
        kind: IPAddressPool
        metadata:
            name: first-pool
            namespace: metallb-system
        spec:
            addresses:
            - 192.168.255.200-192.168.255.250  # kind 네트워크에 맞게 수정
        ---
        apiVersion: metallb.io/v1beta1
        kind: L2Advertisement
        metadata:
            name: my-l2-advertise	#L2Advertisement의 이름
            namespace: metallb-system
        spec:
            ipAddressPools:	#광고할 IP 주소 풀의 이름
                - first-pool


    #### IP Pool 을 지정할 때 유의할 점 (GPT 피셜 정확하지 않을 수 있음)

    * kind 네트워크(docker network inspect kind)와 겹치지 않도록 설정

    * 호스트 네트워크(ip a)와 충돌하지 않도록 설정

    * Pod 네트워크(10.244.0.0/16 등)와 겹치지 않도록 설정

    * LoadBalancer 서비스가 외부에서 접근할 수 있도록 적절한 IP 대역 선택

    * 넉넉한 IP 범위를 설정 (예: 192.168.100.100-192.168.100.200)

    * L2Advertisement 설정을 추가해야 함

    아마 metallb 이 설치된 클러스터 내부에서 임의의 서브넷을 하나 더 만드는 듯 함.

3. 생성한 metallb-config 적용

        kubectl apply -f metallb-config.yaml
    
    >  적용된 결과

        (base) root@djlee:/home/djlee/projects/k8s/01_install_metallb# kubectl get ipaddresspools -n metallb-system
        NAME         AUTO ASSIGN   AVOID BUGGY IPS   ADDRESSES
        first-pool   true          false             ["192.168.255.200-192.168.255.250"]
        
        (base) root@djlee:/home/djlee/projects/k8s/01_install_metallb# kubectl get l2advertisements -n metallb-system
        NAME              IPADDRESSPOOLS   IPADDRESSPOOL SELECTORS   INTERFACES
        example                                                      
        my-l2-advertise   ["first-pool"] 