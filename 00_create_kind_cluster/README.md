# Kind 를 활용하여 k8s 클러스터 구성

> https://kind.sigs.k8s.io/   


1. kind 는 yaml 파일을 통해 클러스터 환경을 구성할 수 있다.

    #### kind_cluster.yml 파일 생성

        kind: Cluster
        apiVersion: kind.x-k8s.io/v1alpha4
        networking:
        apiServerAddress: "0.0.0.0"         # 외부에서 접속 가능하게 하기 위해
        apiServerPort: 56443                # control-plane 컨테이너의 k8s api를 개방
        nodes:
        - role: control-plane
            image: kindest/node:v1.32.3
        - role: worker
            image: kindest/node:v1.32.3
        - role: worker
            image: kindest/node:v1.32.3
        - role: worker
            image: kindest/node:v1.32.3

2. 클러스터 생성

        kind create cluster --name temp --config kind_cluster.yml

    #### 명령어 실행 결과

        (base) root@djlee:/home/djlee/projects/k8s/00_create_kind_cluster#  kind create cluster --name temp --config kind_cluster.yml
        Creating cluster "temp" ...
        ✓ Ensuring node image (kindest/node:v1.32.3) 🖼
        ✓ Preparing nodes 📦 📦 📦 📦  
        ✓ Writing configuration 📜 
        ✓ Starting control-plane 🕹️ 
        ✓ Installing CNI 🔌 
        ✓ Installing StorageClass 💾 
        ✓ Joining worker nodes 🚜 
        Set kubectl context to "kind-temp"
        You can now use your cluster with:

        kubectl cluster-info --context kind-temp

        Have a nice day! 👋

    #### 생성된 k8s 클러스터 컨테이너 확인

        docker ps -a

        (base) root@djlee:/home/djlee/projects/k8s/00_create_kind_cluster# docker ps -a
        CONTAINER ID   IMAGE                  COMMAND                  CREATED              STATUS                    PORTS                     NAMES
        6af023e8c523   kindest/node:v1.32.3   "/usr/local/bin/entr…"   About a minute ago   Up About a minute         0.0.0.0:56443->6443/tcp   temp-control-plane
        6d0de44425ee   kindest/node:v1.32.3   "/usr/local/bin/entr…"   About a minute ago   Up About a minute                                   temp-worker
        5f04f0fa5519   kindest/node:v1.32.3   "/usr/local/bin/entr…"   About a minute ago   Up About a minute                                   temp-worker2
        07706d25a87c   kindest/node:v1.32.3   "/usr/local/bin/entr…"   About a minute ago   Up About a minute                                   temp-worker3
        792405705f2f   wolpython-server       "python main.py"         7 days ago           Up 7 days                                           wolpython-server-1
        a7c74be5ff32   dockurr/windows        "/usr/bin/tini -s /r…"   9 days ago           Exited (143) 4 days ago                             windows

3. kind cluster 생성 시, ~/.kube/config 에 자동으로 context 정보가 주입됨.

    #### .kube/config

        apiVersion: v1
        clusters:
        - cluster:
            certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJSFpnVVRMOHByMDB3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TlRBek1qTXhNekV6TWpKYUZ3MHpOVEF6TWpFeE16RTRNakphTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUURRV2ZOYVJncHlBSnBYZXlQWHhCUWJIdEx4T0xRMHpqc2UwbnlEemxPZTFieFdSaUVKQXF5Yy9OdDAKMnpweFVJNFRvcVJ6MmU4MkE0TjJad3hSVERoOVVYd1RzWjhNeEgwOU5tbWFkUzV1ZEM4UUk3cGh6eDkzS29lKwpoU0lxVmlnK1ZNaDRtUG1qYzA2b05lMjlveEJiUlI4ZzZrK1hIekJKWVoxbStXY3JLMlArVjhISFJma1krSDY4CkRvRUtIbFhtQUp1UlRWMkhsUXl4ekJIOUVCRjlFUERqald2K3FzVXVSSU1zTE1mUkl4SjVLbk9LSjRoVzQ2UmwKTU9QVUdaTFBRNXlNbEpFSDd4TGVOMW9Tcm9mYlA0OStrckUyVTJuRVBZTngvWDRFSE9la09HNHRpdzJRT0QvTgpRWFRqNWh3ZHhhUGhZVTh0N0dURk5lQVpWRkxEQWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJUOE1SaU94elpRSW5OU2lpSFE3R3lSRk45bllUQVYKQmdOVkhSRUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQmN4NkRYUnVmagpYaTBvdXF1ZC9lQkRERXJ6a0h5bjBNWjVMM1pRQ0w1MWl6OWhrc0hWSU4xK1lyVnNkU3dKdGFjNjBmQWxJUndLClpIWEpVWmd2TlplVzl6TnAwWS90M3dxZVpJWXo4L2hoeTdqVU9NMy9nekRadGJDVE5kd28rT3hvRzFNYXM1Z0sKQXNPTkJkczNvSE93VmJqaVhqWW0zektVQVZsdCtmSkV6OXV0RHB5b2tZR2tQWVBhYTRuYTc1Qm9aK2thNFVqNAplYkFTTy9PQlcyWStReFFiZ3QrdGVmNWJhWWlrelozbFVpUFJqTXFlVnR3VmxZdC9BRjdSbzMyRFc1UkVObTlTCnFDanZLeEdkZU9JWkZneUp0SEVUTndySDhUQ3NpRm5GUTJheUVIMVUzbjBTVDh0QXJ4UzFvOFlicE1FWVplckoKdFdLNk5yTmk2UmNoCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
            server: https://0.0.0.0:56443
        name: kind-temp
        contexts:
        - context:
            cluster: kind-temp
            user: kind-temp
        name: kind-temp
        current-context: kind-temp
        kind: Config
        preferences: {}
        users:
        - name: kind-temp
        user:
            client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURLVENDQWhHZ0F3SUJBZ0lJYXR3RjFRSzhyZkF3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TlRBek1qTXhNekV6TWpKYUZ3MHlOakF6TWpNeE16RTRNakphTUR3eApIekFkQmdOVkJBb1RGbXQxWW1WaFpHMDZZMngxYzNSbGNpMWhaRzFwYm5NeEdUQVhCZ05WQkFNVEVHdDFZbVZ5CmJtVjBaWE10WVdSdGFXNHdnZ0VpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCQVFERFljdTMKVnFtS05aZlU0VXJJU1dvek0xRG1VcnNwOVQzU1ZKM2Z5ZmNUVnhnTHd2Nm5BR1pRMGlvUFV3Z0hWeEVxS2t0Ugo3WGhIZGNuMFF0d1RJZStHU2tqQWVaVUpNbGZsbit5M0JOK0Z0QzlnV3NHaTY5SitWZHo1ZmFEb1BKbUwwQkY1CkJGZ3ZjOEs3aXBlZjJ4SGk5Sy9NR21hRkluMUVud0dML1JRTGVmVm1wWElhQ3RyZm9Fb0RsNnJ6MXZqTzJLaTMKOEtETnhFajRpTERZTTJzZldSM3NVRmhuWVRGVlV0dmRkenpTZW0xTEE4aHpzZ3dYQjhLWGVQc0F6SElUdEs0OQpteWREckdhaTFYL3MzdXJVL29DeEExNU9LR3dmejJ4RGdWeHdGU3JqYmNwTGw1SGFSUkpyby9JRE5rY3JHeVlxCisybmZiTGtvVi82NmtTU2JBZ01CQUFHalZqQlVNQTRHQTFVZER3RUIvd1FFQXdJRm9EQVRCZ05WSFNVRUREQUsKQmdnckJnRUZCUWNEQWpBTUJnTlZIUk1CQWY4RUFqQUFNQjhHQTFVZEl3UVlNQmFBRlB3eEdJN0hObEFpYzFLSwpJZERzYkpFVTMyZGhNQTBHQ1NxR1NJYjNEUUVCQ3dVQUE0SUJBUUF4eDdwT2t4ZzlpRVhadmQ5TGpGa2pDVzVNCjhEbFBuWUxsSWJuZ2t3L25ueEZBRnV4UDhPc0xMS2pBaDRmYnBybm8rNVJtYmJHM3dVYXVxbjAzL1B3ZzRyS08KRE9jYWxxdkVTWDFXaHphaFNEUmFsQ2MyVjZQRWFPcHVuUnJZNFI1Zkx4TFBEWVhMaEFhaHNEc1cxQUtJVERIagpuMDduM2JBck9lazN0aDJhUkNURWJNTHF6Umh3Z1NaandXSmt4cXNETSs4TXptSnJIeFE2NlpoVUNWT2NieDIyCkdRNm90cTUyVnYxNVdEZ1VWUmhnUGhyZ2R2RzJjVXk4aUwxamdyWVk4YkdUYzJLS25oa0xxdWduT2dxQ2FYS04KRm5neVVLdGFDWkZoWXFIQ1QrZnNuam9UeS9WdFFrYjZaa3hESVQ5UEI0eGdSNWF4MG9kKzJOdWdiTGd3Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
            client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBdzJITHQxYXBpaldYMU9GS3lFbHFNek5RNWxLN0tmVTkwbFNkMzhuM0UxY1lDOEwrCnB3Qm1VTklxRDFNSUIxY1JLaXBMVWUxNFIzWEo5RUxjRXlIdmhrcEl3SG1WQ1RKWDVaL3N0d1RmaGJRdllGckIKb3V2U2ZsWGMrWDJnNkR5Wmk5QVJlUVJZTDNQQ3U0cVhuOXNSNHZTdnpCcG1oU0o5Uko4QmkvMFVDM24xWnFWeQpHZ3JhMzZCS0E1ZXE4OWI0enRpb3QvQ2d6Y1JJK0lpdzJETnJIMWtkN0ZCWVoyRXhWVkxiM1hjODBucHRTd1BJCmM3SU1Gd2ZDbDNqN0FNeHlFN1N1UFpzblE2eG1vdFYvN043cTFQNkFzUU5lVGloc0g4OXNRNEZjY0JVcTQyM0sKUzVlUjJrVVNhNlB5QXpaSEt4c21LdnRwMzJ5NUtGZit1cEVrbXdJREFRQUJBb0lCQUdGeDd4WjdoSWRILzNmTwovV3N6SW1KeTM1QmdCclVBZVZyamxQRytXeG9zUC9QdHh2QW54Ti9lVWRmZXc0eFZvbHZ6U0NtT1ZJVGZmRi8wCjBLcENMS0kvZmxWd3ppSU9GOFNRcEpFTFB5Z0NHL2Jrak5yaTN0TGZwQnhTeWVQS0JaS3pyV004QlhkMU50UXUKWlR6M0Y4Nm4xdDNtOU9iRnN0Qjh0VnJLV0NyNFF6eFZoVVlodWYvSTFIdkJQQXFFQ1h6RkxCdWZVNjVrcUhhVApkWXJvKzlWTFhzcmN4b0J2T0JkMitWYjZPWjhEV0NRYVhVWEpmWW5YdG1zNGsrVzVNM1ZwRytmNVlzRmRxNk45Ck0zTG54K1RzYkRxMnVnZ2ZhY0M2dTJJaE16dS9LTmtBUDR1M0w2K250bkZwVVFWd2VWZXdRN0R3YkFRdi9GNzAKWmpCdW5YRUNnWUVBN0diUTBWckFFRklnMmlxRjU5b0htQTcxZCtWQVRWZld3a3U4Y3JYTHJUZnpIakFpVGNCYwplOU1ESDk2TDVzdncyUCs1cmdGN2ZLTEI0MzRqWkVXVk9MV21hSTFYTFJjMkdKaElUN0ZiMkNKUjBMLy9oMk9wCkVpd1RINEswLzF4ODEwTlhmMU8rVGR5UnBpaTZSY0UxaTVHeTlpeEt6SXdaOEhyNUpsbFJ2Q2NDZ1lFQTA1UnEKVFdxYWlXQzVHRC9JSUZZSnppeGZYcDV2KzZJNU9YVGNrRnBXaFdLaWlsaUREbUw5ci9WalV1KzJkdlVvZzgzVQp1SDhwVkZPNmhVM3hURDdsWXBNN005YXAvaEYwSnFiL3U3eXJXdWVGNk1zcHJhcGZZYmh0VmZEMkRRNXdQTUVkClFsWmtYa3pmV0RQUWUwRVpLYStiTWVtQzBoUG1KZmRBb3IwNHVHMENnWUVBclNrK3lpdEVSbkF5T2p3dHE4QU0KRWZqYkcwQ2swa0tHUC9vRUJxNWRpL3RRclFzckJYTGpDNXhzVEhyU1ZYT0xieGdhWlhnV2dSd2pFOFZBbldGTwp4YVJoU1hKR3FmTzNuMXBrbFdOZjJEaURYM3BUN1ZNMTgrYXU5MFRoMmE5Z2pybDRMUDhsaFprTVl2NndPd29rCmM2Qjh5MDkwVnRKRTZkN2FBNW9uZ2I4Q2dZQTJTZTY4em8yNGtranNIL0dKMm9uSmpUa3JYaHY5eFRKSnUrS2MKWjVHcnlCTk16RWxVZDdJQVpFYUlFVm9RUy9lSldsY3F2L1lxM1JFUEEyRFczNHljTG9zU1VoSnNUcTR1L01yQgpzVGVHcThHQWFpRFhucys2azBmNnRVbHRNRGM4WDVEU1pMaDhPZDFWRkhaNktjbjdHRVFLR3BDbXR0Um5DWHBjClI1RTJRUUtCZ1FEbGVzYXNFcWUvNEpkcDkrV0dPMFF4c0w3d0U1L1B3Q29HcmsvTnVNcXFueHVMMm4yUjlsaGcKazk4aUhtcWFWbGg2NWttVHhtOTk3QUcrWWU2OG1GMjJuVDQycDB4ZWc5L1Z2SmZjZ3AvSFNoY1UxYVEvNVo0ZwplUkE0ZEJ6YWMwUEErZmRhUWJCZ1FoUmVlSHQrKzhzV0Y1MDJXYSszS0dZS2tQVGMzYzZwSHc9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
        
    파일의 적혀있는 인증서와 키 정보는 kind 자체적으로 생성한 것이라,    
    외부에서는 일반적으로 접근은 되지 않으나, 해당 파일을 연결하는    
    클라이언트의 .kube/config 파일에 override 하고 ssh -L 옵션을 통해    
    터널링을 생성하면 외부에서도 연결할 수 있다.   
