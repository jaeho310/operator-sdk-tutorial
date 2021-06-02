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

operator sdk를 통해 crd, cr, cc를 구축하여 간단한 웹 서비스를 관리하는 예제입니다.  
아래와 같은 cr을 쿠버네티스 api server에 요청하면
<details><summary>cr</summary>
<p>

```yml
apiVersion: mygroup.example.com/v1
kind: Hello
metadata:
  name: hello-sample
  namespace: default
spec:
  size: 3 # cr의 size라는 field로 pod의 갯수를 제어하겠습니다.
```

</p>
</details>

아래의 deployment와 service를 생성한것처럼 동작할 것입니다.(replicas를 제외한 나머지는 모두 하드코딩)
<details><summary>deployment, service</summary>
<p>

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echoservice-dp
  namespace: jh
spec:
  replicas: 3 # cr의 size 입니다. 
  selector:
    matchLabels:
      app: echoservice
  template:
    metadata:
      labels:
        app: echoservice
    spec:
      containers:
        - name: echoservice
          image: repo.iris.tools/test/echoproject:4
```

```yml
apiVersion: v1
kind: Service
metadata:
  name: echoservice-np
  namespace: jh
spec:
  type: NodePort
  ports:
    - port: 8375 
      protocol: TCP 
      targetPort: 8395 
      nodePort: 30012 
  selector: 
    app: echoservice
```

</p>
</details>


### 개발방법
---
### 1. operator sdk를 설치합니다.
<details><summary>click</summary>
<p>

[operator-sdk 설치](https://sdk.operatorframework.io/docs/building-operators/golang/installation/)

```bash
# 해당 명령어를 입력했을때 version이 정상적으로 출력되면 설치완료 
operator-sdk version  
```

</p>
</details>

### 2. operator-sdk init 명령어를 통해 초기화합니다.
<details><summary>click</summary>
<p>

```bash
operator-sdk init --domain=example.com --repo=github.com/my/tutorial 
```

</p>
</details>

### 3. api와 controller를 생성합니다.
<details><summary>click</summary>
<p>

```bash
operator-sdk create api --version=v1 --kind=MyProject --group=jhgroup

## 아래와 같은 질문이 올라오면 모두 y 해줍니다.
Create Resource [y/n]
y
Create Controller [y/n]
y
```

</p>
</details>


### 4. golang을 이용해 crd를 정의합니다.
<details><summary>click</summary>
<p>
api/{version} 아래에 위치한 {kind}_types.go 파일을 수정합니다.<br>
추후 make manifest라는 명령어를 사용하면 해당 go 파일로 정의한 crd의 yaml파일이 생성됩니다.

```bash

```

</p>
</details>

### 5. golang을 이용해 controller를 커스터마이징합니다.

<details><summary>click</summary>
<p>
/controllers 아래에 위치한 {kind}_controller.go 파일을 수정해서 Controller의 로직을 직접 구현합니다.

```bash

```

</p>
</details>

### 6. type파일 상태 업데이트, crd yaml파일 생성, api server에 crd 반영

<details><summary>click</summary>
<p>

type파일 상태를 업데이트합니다.

```bash
make generate
```

crd yaml파일을 생성합니다.
config/crd/bases 아래에 crd yaml파일이 생성됩니다.
```bash
make manifests
```

crd 적용
```bash
make install
```

crd 적용 확인 {kind}.{group}.{domain} 으로 crd가 적용되어 있습니다.
```bash
kubectl get crd
kubectl explain {kind}
```

</p>
</details>

### 7. operator 실행(controller 동작)
<details><summary>click</summary>
<p>


```bash
make run
# or
go run main.go
# or
debugging!!!!!!!!!!!
```

</p>
</details>

### 8. cr 작성 및 apply
<details><summary>click</summary>
<p>

crd/samples 아래에 {group}_{version}_{kind}.yaml 파일에 apiversion이 적용되어있습니다.<br>
crd를 참고하여 작성하고 apply하여 쿠버네티스가 원하는대로 동작하는지 확인합니다.

</p>
</details>