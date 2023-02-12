# 컨피그맵과 시크릿: 애플리케이션 설정

## 7.1 컨테이너화된 애플리케이션 설정
컨피그맵은 설정 데이터를 최상위 레벨의 쿠버네티스 리소스에 저장하는 방법을 말함.</br>
시크릿은 컨피그맵과 비슷한 설정 데이터를 저장하지만 암호화 키 값은 보안과 관련된 정보를 저장한다는 차이가 존재.

## 7.2 컨테이너에 명령줄 인자 전달

### 7.2.1 도커에서 명령어와 인자 정의
dockerfile에서 ENTRYPOINT는 컨테이너가 시작될 때 호출될 명령어를 정의하고 CMD는 ENTRYPOINT에 전달되는 인자를 정의.</br>
</br>
ENTRYPOINT를 전달하는 형식은 shell과 exec형식이 있는 shell은 ex) ENTRYPOINT node app.js 방식이고 exec 방식은 ex) ENTRYPOINT ["node","app.js"] 방식이다. shell 방식의 경우 shell 프로세스로 메인 프로세스가 실행되기 때문에 node 프로세스로 실행되는 exec 형식을 이용해서 주로 실행한다.
</br>

### 7.2.2 쿠버네티스에서 명령과 인자 재정의
쿠버네티스에서 컨테이너를 정의할 때 도커와 마찬가지로 둘다 쓸 수 있는데 여기서는 command와 args 속성을 이용한다.
``` yaml
kind: Pod
spec:
    containers:
    - image: some/image
      command: ["/bin/command"]
      args: ["arg1","arg2","arg3"]
```
args로 인자를 전달할 수도 있음.
``` yaml
apiVersion: v1
kind: Pod
spec:
    containers:
    - image: some/fortune:args
      args: ["15"]
      name: html-generator
      volumeMounts:
      - name: html
        mountPath: /var/htdocs
```
some/fortune:args 이미지의 도커파일
``` Dockerfile
FROM ubuntu:latest
RUN apt-get update; apt-get -y install fortune
ADD fortuneloop.sh /bin/fortuneloop.sh
ENTRYPOINT ["/bin/fortuneloop.sh"]
CMD ["10"]
```
아래는 fortuneloop.sh 파일
``` sh
#!/bin/bash
trap "exit" SIGINT
INTERVAL=$1
echo Configured $INTERVAL time.
mkdir -p /var/htdocs
while :
do
    echo $(date) writing fortune to /var/htdocs/index.html
    /usr/games/fortune > /var/htdocs/index.html
    sleep $INTERVAL
done
```

## 7.3 컨테이너의 환경변수 설정
컨테이너화된 애플리케이션은 종종 환경변수를 설정 옵션의 소스로 사용하는 경우도 있음.
아래는 소스를 이용한 환경변수 설정 fortuneloop.sh 파일과 새로운 yaml파일
``` sh
#!/bin/bash
trap "exit" SIGINT
echo Configured $INTERVAL time.
mkdir -p /var/htdocs
while :
do
    echo $(date) writing fortune to /var/htdocs/index.html
    /usr/games/fortune > /var/htdocs/index.html
    sleep $INTERVAL
done
```
``` yaml
kind: Pod
spec:
    containers:
    - image: some/image
      env:
      - name: INTERVAL
        value: "30"
      - name: CP_INTERVAL
        value: $(INTERVAL)
      name: html-generator
      
```
위처럼 다른 환경 변수를 참조할 수도 있음. 하지만 환경변수를 value 필드로 하드코딩 하는 것보단 컨피그맵같은 리소스를 이용해서 설정을 분리하는게 좋음. 컨피그맵 같은 리소스를 이용할땐 value대신 valueFrom을 사용! </br>

## 7.4 컨피그맵으로 설정 분리
컨피그맵은 key/value 쌍으로된 Map형태임. 컨피그맵의 내용은 컨테이너의 환경변수나 볼륨 파일로 전달됨.

#### kubectl을 이용한 configmap 만드는 방법
아래는 fortune-config의 이름을 가진 configmap하나를 만들고 key/value로 INTERVAL/25를 가진다.</br>
`kubectl create configmap fortune-config --from-literal=INTERVAL=25`</br>
여러 개를 만들고 싶을 경우엔 뒤에 --from-literal만 계속 추가해주면 됨.</br>
이렇게 key/value를 직접 넣을 수도 있지만 파일 내용도 컨피그맵을 생성해줄 수 있고 디렉토리에 있는 파일도 가능.</br>
환경변수를 컨피그맵에서 가져오는 쿠버네티스의 yaml
``` yaml
kind: Pod
spec:
    containers:
    - image: some/image
      env:
      - name: INTERVAL
        valueFrom:
            configMapKeyRef:
                name: fortune-config
                key: INTERVAL
```
``` sh
kubectl create configmap my-config
    --from-file=foo.json   //foo.json라는 키로 내용이 value로 들어감
    --from-file=bar=foobar.conf // bar라는 키로 conf파일의 내용이 value로 들어감
    --from-file=config-opts //해당 디렉토리에 있는 파일명이 key로 내용은 value로 들어감
    --from-literal=some=thing
```
env 속성 대신 envFrom 속성을 사용하면 환경변수로 컨피그맵의 내용을 전부 노출할 수 있음.
``` yaml
spec:
    containers:
    - image: some-image
      envFrom:
      - prefix: CONFIG_ //모든 환경변수가 COFIG_라는 접두사를 가지게됨.
      configMapRef:
        name: my-config
```

볼륨파일로 컨피그맵 항목을 파일로 노출하는 방법도 존재. 아래는 nginx.conf파일
``` conf
server{
    listen 80;
    server_name www.test.com;
    gzip on;
    gzin_types text/plain
    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}
```
</br>

`kuberctl create configmap fortune-config --from-file=temp`
</br>

temp디렉토리에 이 파일과 sleep-interval이라는 텍스트 파일이 있을 때 이걸로 fortune-config라는 configmap을 만들고 쿠버네티스 팟에 마운팅하는 yaml파일
``` yaml
apiVersion: v1
kind: Pod
spec:
    containers:
    - image: nginx:alpine
      name: web-server
      volumeMounts:
      - name: config
        mountPath: /etc/nginx/conf.d
        readOnly: true
    volumes:
    - name: config
      confingMap:
        name: fortune-config
```
이렇게 하면 /etc/nginx/conf.d 디렉토리에 파일이 마운팅 된다. 만약 특정 컨피그맵 항목만 노출하고 싶다면 다음처럼 작성
``` yaml
apiVersion: v1
kind: Pod
spec:
    containers:
    - image: nginx:alpine
      name: web-server
      volumeMounts:
      - name: config
        mountPath: /etc/nginx/conf.d
        readOnly: true
    volumes:
    - name: config
      confingMap:
        name: fortune-config
        items:
        - key: nginx.conf
          path: gizp.conf
```
이렇게 파일을 마운팅할 경우 리눅스의 경우 마운트한 파일시스템에 있는 파일만 포함하고 원래 있던 파일은 해당 파일시스템이 마운트 되어있는 동안 접근이 안됨. 쉽게 말해 /etc에 마운트 하게 되면 기존에 있던 파일들은 못쓰게 돼서 시스템이 정상 작동을 안할 수도 있음. 그래서 속성에 subPath 속성을 이용하면 전체 볼륨을 마운트하는 대신 일부나 마운트 할 수 있음.
``` yaml
apiVersion: v1
kind: Pod
spec:
    containers:
    - image: nginx:alpine
      name: web-server
      volumeMounts:
      - name: config
        mountPath: /etc/nginx/conf.d/gzip.conf
        subPath: nginx.conf
    volumes:
    - name: config
      confingMap:
        name: fortune-config
        items:
        - key: nginx.conf
          path: gizp.conf
```
컨피그맵의 볼륨은 주로 파일 권한이 644로 설정되는데 이는 defaultMode를 통해 변경 가능.</br>
</br>
컨피그맵의 내용을 업데이트하면 참조하는 모든 볼륨의 파일이 업데이트되는데 팟 안에 있는 컨피그맵을 참조하는 프로세스는 리로딩이 되지 않는 이상 내용이 업데이트 되지 않음. 그리고 전체 볼륨이 아닌 단일 파일을 컨테이너에 마운트한 경우 파일이 업데이트 되지 않음. 쿠버네티스는 심볼릭 링크를 이용해 마운트된 컨피그맵 볼륨의 모든 파일을 업데이트하게 된다. 하지만 컨피그맵 볼륨의 파일이 실행 중인 모든 인스턴스에 걸쳐 동기적으로 업데이트 되지 않기 때문에 개별 파일의 내용이 달라지는 현상이 일어날 수도 있음.

## 7.5 시크릿으로 민감한 데이터를 컨테이너에 전달
시크릿은 컨피그맵과 마찬가지로 key/value를 가진 map 형태. 하지만 시크릿은 항상 노드의 메모리에 저장되게 하고 물리적인 저장소에 저장하지 않음. 시크릿을 생성하기 앞서 개인 키 파일과 인증서를 만듬. </br>
`openssl genrsa -out https.key 2048`</br>
`openssl req -new -x509 -key https.key -out https.cert -days 3650 -subj /CN=www.test.com`</br>
`ehco test > test`</br>
`kubectl create secret generic fortune-https --from-file=temp/`</br>
시크릿의 유형에는 세 가지가 존재하는 도커 레지스트리를 사용하기 위한 경우엔 docker-registry, tls통신을 위해선 tls,generic이 있음.</br>

컨피그맵의 경우 데이터는 문자열이지만 시크릿 항목의 데이터는 Base64 인코딩 문자열로 되어있어서 바이너리 값도 담을 수 있음. 하지만 시크릿의 최대 크기는 1MB로 제한되기 때문에 민감한 데이터만 넣음. 텍스트로 데이터를 쓰고 싶을 경우 yaml파일을 작성할 때 stringData필드로 설정할 수 있음. 하지만 이건 쓰기 전용이라서 kubectl get으로 가져올때 해당 필드가 표시되지 않음.</br>
nginx에서 https로 통신을 할 경우 /etc/nginx/certs경로에 시크릿을 마운팅 해주면 됨.
``` yaml
  volumes:
    - name: config
      confingMap:
        name: fortune-config
        items:
        - key: nginx.conf
          path: https.conf
    - name : certs
      secret:
        secretName: fortune-https
```
secret볼륨은 시크릿 파일을 저장하는데 인메모리 파일시스템(tmpfs)을 사용하는데 그 이유는 시크릿의 데이터는 보안과 관련된 민감한 데이터이기 때문에 디스크에 저장하지 않기 위해서이다. 시크릿 키 또한 환경 변수로 노출 시킬 수 있지만 로그에 환경변수가 남을 수도 있어서 좋은 방법은 아님!
``` yaml
kind: Pod
spec:
    containers:
    - image: some/image
      env:
      - name: INTERVAL
        valueFrom:
            secretKeyRef:
                name: fortune-https
                key: test
```
도커 허브에서 프라이빗 이미지를 사용할 때도 이 시크릿 오브젝트를 이용할 수 있음.</br>
``` exec
kubectl create secret docker-registry dockerlogin \
    --docker-username=myname --docker-password=mypwd \
    --docker-email=test@email.com
```
프라이빗 이미지를 이용하는 yaml파일
``` yaml
apiVersion: v1
kind: Pod
spec:
    imagePullSecrets:
    - name: dockerlogin
    containers:
    - image: user/private
      name: test
```
물론 매번 지정할 필요없이 시크릿을 ServiceAccount에 추가해서 모든 파드에 자동으로 추가되게 할 수도 있음!

</br>
!!참고로 코드가 최신 버전이 아닌 것도 있지만 나중에 바꿀 예정