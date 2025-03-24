# 클러스터에 ingress 설치
> 참고 : https://velog.io/@dhkim1522/k8s-%EC%9D%B8%EA%B7%B8%EB%A0%88%EC%8A%A4-%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC-ingress-controller-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0
1. Ingress 설치

        kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.6.4/deploy/static/provider/cloud/deploy.yaml


2. Ingress 설정 (yaml)

     * ingress-sample.yml

                apiVersion: networking.k8s.io/v1
                kind: Ingress
                metadata:
                name: ingress-nginx
                annotations:
                nginx.ingress.kubernetes.io/rewrite-target: /
                kubernetes.io/ingress.class: "nginx"
                spec:
                rules:
                - host: "ingress.test.com"
                http:
                paths:
                - pathType: Prefix
                        path: /test
                        backend:
                        service:
                        name: service-test
                        port:
                        number: 80

	> kubectl apply -f ingress-sample.yaml
	
3. Service Sample 

        # svc-test.yaml

        apiVersion: v1
        kind: Service
        metadata:
        name: service-test
        spec:
        selector:
        app: service_test_pod
        ports:
        - protocol: TCP
        port: 80
        targetPort: 8080

4. Deploy Sample

		# deploy-test.yaml

		kind: Deployment
		apiVersion: apps/v1
		metadata:
		name: service-test
		spec:
		replicas: 3
		selector:
			matchLabels:
			app: service_test_pod
		template:
			metadata:
			labels:
				app: service_test_pod
			spec:
			containers:
			- name: simple-http
				image: python:2.7
				imagePullPolicy: IfNotPresent
				command: ["/bin/bash"]
				args: ["-c", "echo '<p>Hello from $(hostname)</p>' > index.html; python -m SimpleHTTPServer 8080"]
				ports:
				- name: http
				containerPort: 8080