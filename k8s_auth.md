## 1. 쿠버네티스 인증이란
- 쿠버네티스 사용자가 맞는가를 확인하는작업

## 2. 쿠버네티스 api서버의 인증방법
- basic auth
- jwt

```bash
쿠버네티스 api서버는 인증방식으로 basic auth와 jwt를 지원합니다.

아래의 명령어를 통해 token을 확인할수 있으며 http header에 넣어 api서버에 접근할 수 있습니다.
$ kubectl describe secret $(kubectl get secrets | grep default | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d '\t'

api 서버의 url은 아래의 명령어로 확인이 가능합니다.
$ kubectl config view | grep server | cut -f 2- -d ":" | tr -d " "

헤더에 bearer 토큰을 넣어주면 api서버에 접근해 api 목록을 확인할 수 있습니다.
http://<apiserver>/api

해당 클러스터의 pod 목록을 출력해해보면
http://<apiserver>/api/v1/pods
인가가 되지 않아 오류가 발생하는걸 확인할 수 있습니다.
```


## 3. 쿠버네티스 인가
- 사용자에 맞는 권한을 부여하는 작업


