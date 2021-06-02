# operator-sdk
## operator-sdk란
- Operator 개발 플로우를 제공하는 프레임워크

## Operator SDK with Golang
> 주요 패키지
- Runtime Controller package
    - Controller가 관리할 CR의 변경을 감지하고 SDK Controller Package의 Reconcile Loop가 동작하도록 요청
- Runtime Manager Package
    - k8s client 및 Cache를 초기화
- SDK Controller Package
    - 실제 Controller 로직을 수행하는 Reconcile Loop와 Runtime Manager Package로 부터 전달받은 Kubernetes Client를 포함

## 개발방법 및 예제
예제는 operator sdk를 통해 crd를 정의하고 커스텀 컨트롤러를 구축하여
간단한 웹 서비스를 관리하는 예제입니다.
[echoservice.app.yml]
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echoservice-dp
  namespace: jh
spec:
   replicas: 2 
  selector:
    matchLabels:
      app: echoservice
      tier: app
  template:
    metadata:
      labels:
        app: echoservice
        tier: app
    spec:
      containers:
        - name: echoservice
          image: repo.iris.tools/test/echoproject:4
```

[echoservice.nodeport.yml]
```yml
apiVersion: v1
kind: Service
metadata:
  name: echoservice-np
  namespace: jh
spec:
  type: NodePort
  ports:
    - port: 8375 # 서비스가 생성할 port
      protocol: TCP # 프로토콜
      targetPort: 8395 # 컨테이너 포트
      nodePort: 30012 # 외부에서 접근할 port
  selector: # 서비스가 접근할 pod의 label조건
    app: echoservice
    tier: app
```
- operator-sdk를 install 한다.   
    - https://sdk.operatorframework.io/docs/installation/  
$ operator-sdk version  
명령어를 입력했을때 version이 정상적으로 출력되면 설치완료 

- operator-sdk를 init한다 
    - $ operator-sdk init --domain=example.com --repo=github.com/my/tutorial  

- api와 controller를 생성한다.  
    - $ operator-sdk create api --version=v1 --kind=MyProject --group=jhgroup  
- golang을 이용해 crd를 정의해준다.  
    - api/{version} 아래에 위치한 {kind}_types.go 파일을 수정하여 crd를 정의해준다.
추후 manifests라는 명령어를 사용하면 해당go파일에 정의된 crd yaml파일이 생성된다. 
- controller 커스터마이징  
    - ./controllers 아래에 위치한 {kind}_controller.go 파일을 수정해서 Controller의 로직을 직접 구현한다.  
 custom controller(cc)라고하면 컨트롤루프가 정상적으로 돌수있도록 구현합니다.
- make generate, make manifests  
    - make파일을 통해 generate(type파일 상태 업데이트), manifests(crd yaml파일 생성)를 하면 ./config/crd/bases 아래에 crd yaml파일이 생성된다.
- make install  
    - 이제 kubectl get crd 를 해보면 {kind}.{group}.{domain} 으로 crd가 올라가있다.
- make run
    - 실행한다.

- cr 작성 및 apply  
    - 2번 과정 이후에 crd/samples 아래에 {group} _ {version} _ {kind} .yaml 파일이 미리 생성되어있지만
crd에 맞게 커스터마이징 되지는 않으므로 직접 작성한다.
crd를 참고하여 작성하고 apply 하며 나의 operator가 잘 동작하나 확인한다.
