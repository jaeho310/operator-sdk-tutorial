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
# hello라는 Kind의 객체는 쿠버네티스가 뭔지 모르기 때문에 등록이 안됩니다.
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
  # name : 아래의 spec 필드와 맞아야 한다. spec의 plural와 group를 조합한 값이와야 한다. <plural>.<group>
  name: hellos.extension.example.com
spec:
  # REST API 용 그룹 이름: /apis/<group>/<version>
  group: extension.example.com
  # API 버전
  version: v1
  # Namespaced/Cluster 범위 표시
  scope: Namespaced
  names:
    # api URL에서 사용할 복수형 이름 : /apis/<group>/<version>/<plural>
    plural: hellos
    # CLI에서 사용할 단수형 이름
    singular: hello
    # kind에 사용할 단수형 단어.
    kind: Hello
```
해당 yaml파일을 이용해 crd를 생성, api server에 반영이 됐나 확인합니다.
```bash
kubectl apply -f hellocrd.yaml
kubectl explain hello
kubectl get crd
```

</p>
</details>