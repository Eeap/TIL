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