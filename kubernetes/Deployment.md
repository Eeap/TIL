# Deployment
## 9.1 파드에서 실행 중인 애플리케이션 업데이트
파드를 업데이트하는 방법에는 두가지가 존재. 하나는 기존 파드를 모두 삭제한 다음 새 파드를 시작하는 방법, 다른 방법은 새로운 파드를 시작하고 기동하면 기존 파드를 삭제하는 방법이 존재. 첫번째 방법은 블루-그린 디플로이먼트라고 부르고 두번째 방법은 롤링업데이트 방식이라고 불림. 첫 번째 방법의 단점은 기존 파드에서 새 파드로 교체되는 시간 동안에는 애플리케이션을 사용할 수 없다는 단점이 있고 두 번째 방법의 단점으로는 애플리케이션이 동시에 두가지 버전이 존재하게 된다는 단점이 있음.</br></br>
블루-그린 디플로이먼트 방식은 잠시 동안 두배의 파드가 존재하게 되므로 더 많은 리소스를 필요하게 되지만 동시에 두가지 버전이 존재하지는 않음. 그리고 두 방식 보두 또 다른 레플리케이션컨트롤러가 생성되어서 새로운 파드를 생성함.
</br>
</br>
롤링 업데이트의 경우엔 쉽게 kubectl을 이용해서 할 수 있는데 만약 레플리케이션컨트롤러가 k1이 있고 이 image1과 image2 단순히 이미지의 버전만 다르다고 했을 때 `kubectl rolling-update k1 k2 --image=sumin/image2`라는 명령어를 치면 새로운 k2라는 레플리케이션 컨트롤러가 만들어지고 해당 컨트롤러는 image2라는 이미지를 사용해서 파드를 업데이트 한다. 하지만 kubectl을 이용해서 rolling update를 하는 방식은 잘 사용하지 않는데 그러한 이유는 일단 모든 단계를 수행하는 주체가 kubectl 클라이언트 프로세스이기 때문이다. 이는 `kubectl rolling-update k1 k2 --image=sumin/image2 --v 6`를 통해 로깅으로 kubectl클라이언트가 쿠버네티스 마스터 대신 스케일링을 수행하는 것을 확인할 수 있음. 이는 또한 kubectl이 엡데이트하다가 네트워크 연결 같은 상황이 발생하면 프로세스가 중간에 중단될 수 있는 문제가 발생할 수도 있음.
</br></br>
## 9.3 애플리케이션을 선언적으로 업데이트하기 위한 디플로이먼트 사용하기
그래서 애플리케이션을 선언적으로 업데이트하기 위해서는 deployment라는 리소스를 사용하게 되는데 이는 replicaSet과 비슷하긴 하지만 그것보다 더 상위의 개념이다. deployment를 생성하게 되면 그 아래 레플리카셋이 만들어지게 되고 레플리카셋은 해당되는 파드들을 관리하게 된다. 또한 디플로이먼트는 포함된 레플리카셋이 잘 관리되도록 조정하는 역할도 한다. 아래는 deployment를 만드는 yaml파일.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: my-deployment-example
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-deployment-example
  template:
    metadata:
      labels:
        app: my-deployment-example
    spec:
      containers:
      - name: nginx
        image: nginx:v1
        ports:
        - containerPort: 80
      - name: redis
        image: redis
        volumeMounts:
        - name: redis-storage
          mountPath: /data/redis
      volumes:
      - name: redis-storage
        emptyDir: {}
```
`kubectl rollout status deployment my-deployment` 명령을 통해 디플로이먼트의 상태를 확인할 수 있고 describe로 세부 사항을 확인할 수 있다. 디플로이먼트는 파드를 직접적으로 관리하지 않고 그 아래 있는 레플리카셋이 파드들을 관리하게 된다. 레플리카셋의 네임은 디플로이먼트 name에서 뒤에 해당 파드 템플릿의 해시값이 포함된다.
</br>
디플로이먼트의 업데이트 전략에는 Recreate전략과 RollingUpdate라는 전략이 있다. Recreate전략은 한번에 기존 모든 파드를 삭제한 뒤 새로운 파드를 만드는 방식이고 그동안에 서비스 다운타임이 발생하게 된다. RollingUpdate 전략은 이전에 있었던 방식과 동일하게 하나씩 제거하고 동시에 새로운 파드를 추가해서 전체 프로세스에서 애플리케이션을 계속 사용할 수 있도록 하는 방식이다. 하지만 이전과 다른 점은 이전에는 레플리케이션 컨트롤러를 업데이트가 끝나면 삭제했지만 deployment는 삭제하지 않고 놔두는데 그 이유는 새로운 버전에 문제가 생겼을 경우 이전 버전으로 롤백을 해야하기 때문이다.</br></br>

롤링 업데이트의 속도를 느리게 하는 방법도 있는데 디플로이먼트에서 minReadySeconds라는 속성을 이용하면 된다. kubectl patch 명령어로 설정이 가능하기도 하다. 
</br></br>
`kubectl patch deployment my-deployment -p '{"spec": {"minReadySeconds":10}}'`
</br></br>
디플로이먼트의 업데이트를 시작하기 위해선 yaml을 편집해서 apply를 하거나 patch명령어를 이용하는 방법도 있고 set image라는 명령어를 사용해서 업데이트를 할 수도 있다. `kubectl set image deployment my-deployment nginx=nginx:v2`
</br></br>
이렇게 새로운 버전으로 업데이트를 할 수도 있지만 기존의 버전으로 rollback도 가능한데 이전 단계로 되돌리기 위해서는 `kubectl rollout undo deployment my-deployment` 라는 명령어를 입력하면 되지만 특정 버전으로 롤백하기 위해서는 revision history를 보기 위해서 `kubectl rollout history deployment my-deployment` 라는 명령어로 revision 버전을 볼 수 있고 특정 버전으로 롤백하기 위해서는 `kubectl rollout undo deployment my-deployment --to-revision=1` 라는 명령어를 통해 롤백할 수 있다.</br></br>
revision 제한 수는 디플로이먼트의 editionHistoryLimit라는 속성을 통해 제한할 수 있고 기본값은 2로 설정되어 있다. 또 다른 속성으로는 롤링 업데이트 중에 한 번에 몇개의 파드를 교체할지를 결정할 수 있는데 이 역할을 하는게 maxSurge와 maxUnavailable이다.
```yaml
spec:
    strategy:
        rollingUpdate:
            maxSurge: 1
            maxUnavailable: 0
        type: RollingUpdate
```
이렇게 설정하면 되고 maxSurge의 값은 지정한 replica 수보다 얼마나 많은 파드 인스턴 수를 허용할 지 결정하고 만약 replica가 3이면 4까지 허용된다. maxUnavailable은 사용할 수 없는 파드 인스턴스 수를 결정하며 replica가 3이면 항상 3개가 available한 상태가 되어야한다. 이를 이용해서 업데이트의 속도를 조절하는게 가능하다.</br></br>

롤아웃 프로세스를 중지하거나 재개할 수 있는 방법도 있는데 `kubectl rollout pause deployment my-deployment`를 통해서 롤아웃을 일시 정지하고 `kubectl rollout resume deployment my-deployment`를 통해서 다시 재개도 가능하다. 이러한 기능들은 canary release를 효과적으로 실행할 수 있는데 canary release란 잘못된 버전의 애플리케이션이 rollout돼 모든 사용자에게 영향을 주는 위험을 최소화하는 기술을 의미한다. </br></br>

잘못된 버전의 롤아웃을 방지하기 위해선 앞에서 언급했던 minReadySeconds를 이용하는 방법도 있는데 이 속성은 파드를 available한 상태로 만들기 전에 새로 만든 파드를 준비할 시간을 지정하는 값이다. minReadySeconds가 지나기 전에 새 파드가 제대로 작동하지 않고 레디니스 포르브(일정 시간마다 해당 컨테이너에 요청을 날림)가 실패하기 시작하면 새 버전의 롤아웃이 효과적으로 차단된다. 일반적으로 파드는 실제 트래픽을 수신하기 시작한 후 파드가 준비 상태를 계속 보고할 수 있도록 minReadSeconds를 높게 설장한다.</br></br>
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: my-deployment-example
spec:
  replicas: 3
  minReadySeconds: 10
  strategy:
    roliingUpdate:
        maxSurge: 1
        maxUnavailable: 0
    type: RollingUpdate
  selector:
    matchLabels:
      app: my-deployment-example
  template:
    metadata:
      labels:
        app: my-deployment-example
    spec:
      containers:
      - name: nginx
        image: nginx:v1
        ports:
        - containerPort: 80
        readinessProbe:
            periodSecons: 1
                httpGet:
                    path: /
                    port: 80
      - name: redis
        image: redis
        volumeMounts:
        - name: redis-storage
          mountPath: /data/redis
      volumes:
      - name: redis-storage
        emptyDir: {}
```

기본적으로 롤아웃이 10분 동안 진행되지 않으면 실패한 것으로 간주되고 `kubectl describe deployment my-deployment`라는 명령어를 통해 ProgressDeadlineExceeded라는 조건이 표시되면 실패했다는 걸 알 수 있다. 그리고 지정된 시간이 초과되면 자동으로 롤아웃이 중지되게 된다.