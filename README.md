# kubernetes


## 기본 정보 설정

```yaml
apiVersion: v1 # 이 오브젝트를 생성하기 위해 사용하고 있는 쿠버네티스 API ver
kind: Pod # 어떤 종류의 오브젝트를 생성하는지
metadata:	# 이름 문자열, UID, 네임스페이스를 포함하여 유일하게 구분지어줄 데이터
  name: kuard
```



## 쿠버네티스 스팩
https://kubernetes.io/ko/docs/concepts/configuration/manage-resources-containers/

https://kubernetes.io/ko/docs/concepts/storage/volumes/
```yaml
spec:
  volumes:
  - name: "nfs-data" # 볼륨 이름
    nfs: # nfs서버를 사용함(원격 디스크를 사용)
      server: my.nfs.server.local
      path: "/exports"
  - name: "kuard-data"
    hostPath: # 호스트 머신의 경로와 연결시켜줌
      path: "/Users/user/kubernetes" # host의 경로 
  container:
  - image: gcr.io/kuard-demo/ # container를 구성하기
    name: kuard	# 컨테이너 이름
    ports: # 컨테이너가 다음의 포트를 사용할 것이라는 정보 (dockerfile의 EXPOSE와 돌일)
    - containerPort: 8080
      name: http
      protocol: TCP
    resources: # 컨테이너 자원 명시
      requests: # 최소 자원
        cpu: "500m" # 0.5cpu를 최소 cpu
        memory: "128Mi" # 128M를 최소 메모리
      limits: # 최대 자원
        cpu: "1000m" # 1cpu를 최대 cpu
        memory: "256Mi" # 256M를 최대 메모리
    volumeMounts:	# pod에 볼륨을 싣음 
    - mountPath: "/data" # pod에
      name: "kuard-data"
```

## 쿠버네티스 Pod 진단 담당 서비스

https://kubernetes.io/ko/docs/concepts/workloads/pods/pod-lifecycle/

```yaml
spec:
  livenessProbe: # 컨테이너가 동작중? 실패한다면 kubelet은 컨테이너를 죽이고 컨테이너는 재시작 대상이됨
    httpGet:
      path: /healthy
      port: 8080
    initialDelaySeconds: 5
    timeoutSeconds: 1
    periodSeconds: 10
    failureThreshold: 3
  readinesssProbe: # 컨테이너가 준비가 되었는지?/ 컨테이너가 지원하지 않으면 default는 Success
    httpGet:
      path: /ready
      port: 8080
    initialDelaySeconds: 30
    timeoutSeconds: 1
    periodSeconds: 10
    failureThreshold: 3
```


## kubectl 명령어
kubectl apply -f xxxxx.yaml           : xxxxx.yaml 메니페스트를 만듦

kubectl get pods                      : 실행 중인 포드들을 불러옴

kubectl describe pods xxxx            : xxxx이름의 포드에 대한 자세한 정보를 표시함(포드 기본 정보, 포드에서 실행중 컨테이너 정보, 포드 관련 이벤트 정보 순)

kubectl delete pods/xxxx              : xxxx이름의 포드를 삭제

kubectl delete -f xxxx.yaml           : 포드를 생성할때 사용한 파일을 통해서 삭제 할 수도 있음.

kubectl port-forward xxxx 8080:8080   : 포드의 8080 포트를 호스트의 8080으로 포워딩 시킴.

kubectl logs kuard                    : 실행중인 컨테이너의 로그를 가져옴. -f 플래그를 추가하면 스트림으로 받을 수 있고, --previous 플래스를 사용하면 컨테이너의 이전 인스턴스 로그를 가져옴

kubectl exec xxx                      : xxx 포드 내 컨테이너에서 직접 실행을 함.

kubectl exec -it xxx -- /bin/bash     : xxx 포드 내 bash를 통해 대화형 세션을 함.
