# 애플리케이션에서 파드 메타데이터와 그 외의 리소스에 액세스하기

## 8.1 Downward API로 메타데이터 전달
파드의 metadata를 파드 내에서 실행 중인 프로세스인 애플리케이션(컨테이너)에 전달할 때 Downward API를 이용하거나 환경변수를 통해 전달할 수 있음. 보통 다음과 같은 정보를 컨테이너에 전달 </br>
- pod의 이름
- pod의 ip 주소
- pod가 속한 namespace
- pod가 실행 중인 node의 이름
- pod가 실행 중인 ServiceAccount의 이름
- 각 컨테이너의 cpu와 메모리 request
- 각 컨테이너의 cpu와 메모리 제한
- pod의 label
- pod의 annotation

대부분의 항목은 환경변수 또는 downwardAPI 볼륨으로 컨테이너에 전달할 수 있지만 label이나 annotation 같은 경우엔 볼륨으로만 전달이 가능한데 그 이유는 label이나 annotation은 파드가 실행 중인 도중에도 언제든지 바꿀 수 있기 때문이고 그에 반해 환경 변수는 나중에 업데이트를 할 수 없기 때문에 컨테이너에 마운트 할 수 있는 볼륨을 사용.</br>
</br>
아래는 환경변수를 이용한 메타데이터 전달 yaml파일
``` yaml
apiVersion: v1
kind: Pod
metadata:
    name: downward
spec:
    containers:
    - name: main
      image:busybox
      command: ["sleep","99"]
      resources:
        requests:
            cpu: 15m
            memory: 100Ki
        limits:
            cpu: 100m
            memory: 4Mi
      env:
      - name: POD_NAME
        valueFrom:
            fieldRef:
                fieldPath: metadata.name
      - name: POD_NAMESPACE
        valueFrom:
            fieldRef:
                fieldPath: metadata.namespace
      - name: POD_IP
        valueFrom:
            fieldRef:
                fieldPath: status.podIP
      - name: NODE_NAME
        valueFrom:
            fieldRef:
                fieldPath: spec.nodeName
      - name: SERVICE_ACCOUNT
        valueFrom:
            fieldRef:
                fieldPath: spec.serviceAccountName
      - name: CONTAINER_CPU_REQUEST_MILLICORES
        valueFrom:
            resourceFieldRef:
                resource: requests.cpu
                divisor: 1m
      - name: CONTAINER_MEMORY_LIMIT_KIBIBYTES 
        valueFrom:
            resourceFieldRef:
                resource: limits.memory
                divisor: 1Ki
```
컨테이너의 cpu/메모리 요청과 제한은 resourceFieldRef를 사용해 참조하고 리소스 필드의 경우 실제 값은 제수로 나눈 결괏값을 환경변수에 노출함.</br>
`kubectl exec downward env`를 통해 컨테이너에 있는 모든 환경변수를 볼 수 있음.</br>
</br>
환경변수 대신 파일로 메타데이터를 노출하려는 경우엔 downward API 볼륨을 이용하면 됨.
아래는 볼륨을 이용한 메타데이터 전달 yaml파일
``` yaml
apiVersion: v1
kind: Pod
metadata:
    name: downward
    labels:
        foo: bar
    annotations:
        key1: val1
spec:
    containers:
    - name: main
      image:busybox
      command: ["sleep","99"]
      resources:
        requests:
            cpu: 15m
            memory: 100Ki
        limits:
            cpu: 100m
            memory: 4Mi
      volumeMounts:
      - name: downward
        mountPath: /etc/downward
    volumes:
    - name: downward
      downwardAPI:
        items:
        - path: "podName"
          fieldRef:
            fieldPath: metadata.name
        - path: "podNamespace"
          fieldRef:
            fieldPath: metadata.namespace
        - path: "labels"
          fieldRef:
            fieldPath: metadata.labels
        - path: annotations
          fieldRef:
            fieldPath: metadata.annotations
        - path: "containerCpuRequestMilliCores"
          resourceFieldRef:
            containerName: main
            resource: requests.cpu
            divisor: 1m
        - path: "containerMemoryLimitBytes"
          resourceFieldRef:
            containerName: main
            resource: limits.memory
            divisor: 1
```
`kubectl exec downward -- ls -al /etc/downward`를 통해 마운트된 걸 확인할 수 있음. 그리고 컨테이너 수준의 메타데이터를 참조하는 경우엔 containerName을 명시해주어야 하는데 그 이유는 볼륨은 컨테이너가 아닌 파드 수준에서 정의되었기 때문. 따라서 어떤 컨테이너인지 명시해주어여함.

## 8.2 쿠버네티스 API 서버와 통신하기
downward API는 단지 파드 자체의 메타데이터와 모든 파드의 데이터 중 일부만 노출하기 때문에 애플리케이션이 클러스터에 정의된 다른 파드나 리소스에 관한 추가 정보를 얻을 때 도움이 되지 않음. 즉 다른 리소의 정보가 필요하거나 가능한 한 최신 정보에 접근해야하는 경우엔 API서버와 직접 통신해야함.
</br>
kubectl proxy를 이용하면 쿠버네티스 api서버에 직접 접근해볼 수 있음.</br>
`kubectl proxy` 를 한 다음 `curl localhost:8001`로 요청하면 경로 목록을 반환받게 되는데 여기에서 리소스 타입을 확인할 수 있음. 보통 proxy는 8001번 포트로 열림. 해당 그룹에 속하는 리소스를 보고 싶으면 해당 그룹의 path로 curl을 날리면 됨.
예를 들어 apis/batch/v1에 어떤게 있는지 보고 싶으면 `curl http://localhost:8001/apis/batch/v1` 이 api 그룹에는 잡리소스가 포함되어있는데 해당 job리소스의 네임스페이스 지정여부, 잡이 사용할 수 있는 동사 등의 정보가 있음. 만약 여기서 잡의 목록을 더 보고 싶다면 `curl http://localhost:8001/apis/batch/v1/jobs` 명령어를 치면 됨.</br>
잡 목록이 아니라 특정 job에 대해 보고 싶다면 `curl http://localhost:8001/apis/batch/v1/namespaces/default/jobs/job_name`을 치면 되고 결과는 `kubectl get job job_name -o json`과 동일.
</br>
만약 파드 내에서 api 서버와 통신하려면 먼저 api 서버의 위치를 알아야하고 api서버인지 확인이 필요하며 인증 절차가 필요함. pod에서 실행하기 위해선 curl을 날릴 수 있는 컨테이너 하나가 필요함. api서버의 주소는 흔히 아는 `kubectl get svc`로 ip를 알아 낼 수 있고 컨테이너에서 `env | grep KUBERNETES_SERVICE`로 알아낼 수 있음. pod에서는 단순히 kubernetes라는 dns명만으로도 검색이 가능! 하지만 해당 서비스에 접근하려면 https로의 접근이 필요하고 -k옵션을 사용하거나 서버의 인증 절차를 걸쳐서 접근하는 방법이 있음. 각 컨테이너에 자동으로 마운트되어서 시크릿의 내용이 저장되어있음. 경로는 /var/run/secrets/kubernetes.io/serviceaccount/이고 여기에는 인증기관의 인증서를 보유한 ca.crt 파일이 있음. </br>
다음은 인증절차</br>
```
export CURL_CA_BUNDLE=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -H "Authorization: Bearer $TOKEN" https://kubernetes
```

방금 한 절차로 파드가 속한 네임스페이스에 있는 파드를 나열할 수도 있다.
```
NS=$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace)
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/$NS/pods
```
위처럼 파드내에서 애플리케이션으로 쿠버네티스 api에 접근하려면 api서버의 인증서가 인증기관으로부터 서명됐는지 검증해야하고(ca.crt) 토큰 파일의 내용을 http 헤더에 Bearer 토큰으로 넣어 전송해서 자신을 인증해야함. 그리고 namespace 파일은 파드의 네임스페이스 안에 있는 api 오브젝트의 crud 작업을 수행할 때 네임스페이스를 api 서버로 전달하는데 사용.</br>
하지만 이 절차는 보통 복잡해서 proxy처럼 앰배서더 컨테이너를 두고 실제 서비스를 하는 컨테이너가 앰배서더 컨테이너에 curl를 http로 날려서 접근할 수 있음. 앰배서더 컨테이너는 kubectl-proxy 컨테이너 이미지를 기반으로 해서 실행되고 http로 온 요청을 https로 api 서버에 날려서 반환 결과를 받아와서 요청 컨테이너에 돌려주는 역할을 함.</br>
물론 클라이언트 라이브러리를 사용해 api 서버와 통신하는 방법도 있다. 기본적인 python이나 go 등에서 제공되는 라이브러리가 있고 일반적으로 https를 지원하고 인증을 관리해서 앰배서더 컨테이너를 사용할 필요는 없음.