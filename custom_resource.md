# custom resource
## cr(custom resource)
- 쿠버네티스 API에서 특정종류의 API 오브젝트 모음을 저장하는 엔드포인트
- 이 API 확장 오브젝트를 직접 정의해 사용
- k8s 애플리케이션의 추가기능이 필요할때 사용

```
쿠버네티스에서 애플리케이션을 개발하다보면 추가적인 기능이 필요할 수 있습니다.  
쿠버네티스에서는 오브젝트를 직접 정의해 사용할수 있으며 소스코드를 따로 수정하지 않고도
API를 확장해 사용할 수 있는 인터페이스를 제공하고 있습니다.  
```

<details><summary>custom resource 예제</summary>
<p>

``` yaml
apiVersion: "crd.example.com/v1"
kind: Hello
metadata:
  name: hello-sample
size: 3
```
해당 yaml파일을 이용해 kubernetes api server에 요청해봅니다.
```bash
kubectl apply -f hello123.yaml
# error: unable to recognize "hello123.yaml": no matches for kind "Hello" in version "crd.example.com/v1"
# Kind가 hello인 객체는 쿠버네티스가 뭔지 모르는 객체입니다. ->  등록이 안됩니다.
```

</p>
</details>

## crd(custom resource definition)
- CR을 등록하기위해 사용
- 따로 프로그래밍을 하지 않고 yaml만으로 사용 가능
- k8s API서버가 관리

```
커스텀 리소스를 쿠버네티스 etcd에 등록하기 위해서는 CRD(CustomResourceDefinitions)나 AA(API Aggregation) 이용해야합니다.  
AA는 Go를 통해 개발하며 바이너리와 이미지를 따로 만들어줘야하지만 CRD를 사용하면 yaml만으로 CR을 등록할 수 있습니다.
```

<details><summary>custom resource definition 예제</summary>
<p>

```yaml
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: hellos.extension.example.com
spec:
  group: extension.example.com
  version: v1
  scope: Namespaced
  names:
    plural: hellos
    singular: hello
    kind: Hello
```
해당 yaml파일을 이용해 crd를 생성, api server에 반영이 됐나 확인합니다.
```bash
# crd 적용
kubectl apply -f hellocrd.yaml

# crd 확인
kubectl get crd
kubectl explain hello

# cr 생성
kubectl apply -f hello123.yaml

# 생성된 cr 확인
kubectl get hello
```
hello는 만들어지지만 클러스터에는 큰 변화가 없습니다.<br>
hello는 etcd에 저장만 될뿐 동작을 하지 않는 데이터일 뿐입니다.<br>
</p>
</details>
