## 1. 쿠버네티스 인증이란
- 쿠버네티스 사용자가 맞는가를 확인하는작업

## 2. 쿠버네티스 api서버의 인증방법
- basic auth
- jwt

```bash
쿠버네티스 api서버는 인증방식으로 basic auth와 jwt를 지원합니다.

api서버의 url과 token을 사용하여 api서버에 접근할 수 있습니다.
```

<details><summary>http 요청을 통한 인증 확인</summary>
<p>

1. 토큰확인하기
```bash
아래의 명령어를 통해 default serviceaccount의 token을 확인할수 있으며 http header에 넣어 api서버에 접근할 수 있습니다.
$ kubectl describe secret $(kubectl get secrets | grep default | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d '\t'
```

2. api서버 url, port 확인
```bash
api 서버의 url은 아래의 명령어로 확인이 가능합니다.
$ kubectl config view | grep server | cut -f 2- -d ":" | tr -d " "
```

3. api서버에 http request 보내서 인증 확인
(1) anonymous
```
GET 메서드로 https://<apiserver>/api에 접근하면 403에러가 발생합니다.
anonymous 사용자로 인증은 되지만 인가를 해주지 않습니다.
```

(2) 401 에러 확인
```
직접 401 인증에러를 확인하려면 http header에 인증되지 않은 토큰을 넣어 확인할 수 있습니다.
Authorization: Bearer test 를 헤더에 추가하여 https://<apiserver>/api 에 request를 보냅니다.
```

(3) 200 ok
```
헤더에 인증된 bearer 토큰을 넣어 https://<apiserver>/api 에 request를 보내면
쿠버네티스 서버의 api 목록을 확인할 수 있습니다.
```

(4) 403 인가되지않은 사용자
```
클러스터의 pod 목록을 출력하기위해 아래와 같은 request를 보내면
default ServiceAccount에 대한 인가가 되지 않아 오류가 발생하는걸 확인할 수 있습니다.
https://<apiserver>/api/v1/pods
```
</p>
</details>


## 3. 쿠버네티스 인가
- 사용자에 맞는 권한을 부여하는 작업
- RBAC(Role based access control) 방식을 사용


- Service account는 인가를 원하는 사용자의 개념
- role은 watch, get 등 resource에 접근하기위한 규칙을 정의
- rolebinding은 role과 사용자를 연결
- role이나 rolebinding 앞에 cluster가 붙는 경우에는 namespace와 관계없이 resource에 접근 가능
- 반대로 role이나 rolebinding만 사용하면 지정한 namespace에 있는 리소스만 접근 가능

<details><summary>service account를 생성해 인가 구현</summary>
<p>

1. ServiceAccount 생성
```yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
```
2. 클러스터 전체의 pod를 조회할수 있는 ClusterRole 생성
```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: my-cluster-role
rules:
- apiGroups: # ""는 k8s.io/api/core를 나타냅니다.
  - "" 
  resources:
  - pods
  verbs:
  - get
  - list
```
3. ClusterRoleBinding을 통해 service account와 clusterRole 연결
```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: my-cluster-role-binding
subjects:
- kind: ServiceAccount
  name: my-service-account
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: my-cluster-role
```

4. 생성한 ServiceAccount의 토큰을 헤더에 실어 api서버에 request 보내서 확인
```
https://<apiserver>/api/pods
-> 200 ok
https://<apiserver>/api/services
-> 403 forbidden
```

</p>
</details>