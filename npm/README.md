# About node_modules Structure in npm

![](https://velog.velcdn.com/images/adultlee/post/58f1a7c1-da71-4495-86ec-3b8cbfbf7e2b/image.png)

npm 환경에서의 node_modules 의 구조에 대해서 학습합니다.

npm v2 와 npm v3([최신버전 2015년 이후로 stable](https://blog.npmjs.org/post/129378362260/npm-weekly-29-npm-3-out-of-beta-nick-out-of-the))의 주요 변경점에 관하여 정리해보겠습니다.

## NPM v2

npm v2 는 npm v3가 stable 되기 전까지 사용되어 왔던 npm의 과거의 버전입니다.

npm v2에서 의존성 트리를 생성하는 방식을 예를 통해 설명해 보겠습니다. 주어진 상황에서 세 개의 모듈 A, B, C가 있고, A는 B를 v1.0으로 요구하고, C도 B를 요구하지만 v2.0을 요구하는 경우를 가정합니다. 이러한 상황에서 npm v2는 어떻게 의존성 트리를 구성할까요?

상황에 따라 다르겠지만, npm v2는 폴더 내에 각 패키지의 종속성을 각각 설치하는 방식을 취합니다. 따라서 모듈 A, B, C를 설치하려면 각각의 폴더에 각 패키지의 의존성을 설치합니다.

여기서 주의해야할 것이 있습니다. 바로 의존성 지옥(Dependency Hell) 이라고 합니다.

## Dependency Hell

![image](https://github.com/adultlee/study-package-manager/assets/77886826/3c84d7f1-c46d-4fa5-95de-8a998ff1aae1)

Dependency Hell은 소프트웨어 개발 및 관리 과정에서 발생할 수 있는 문제 상황을 나타내는 용어입니다. 이 문제는 의존성 관리 시스템에서 여러 의존성들이 복잡하게 꼬여서 해결하기 어려운 상황을 말합니다. 주로 다음과 같은 상황에서 발생합니다:

1. Version Conflict (버전 충돌): 서로 다른 패키지들이 동일한 패키지에 서로 다른 버전을 요구할 때 발생합니다. 예를 들어, A 패키지가 B 패키지의 v1.0을 요구하고, C 패키지가 B 패키지의 v2.0을 요구할 경우 충돌이 발생합니다.

2. Transitive Dependencies (간접 의존성): 하나의 패키지가 다른 패키지에 의존하며, 다른 패키지가 또 다른 패키지에 의존할 때 발생합니다. 이런 의존성 체인이 복잡하게 꼬일 경우 해결하기 어려운 문제가 생길 수 있습니다.

3. Complex Dependency Graphs (복잡한 의존성 그래프): 여러 개의 패키지들이 복잡하게 얽혀 있을 때, 어떤 패키지를 설치하거나 업데이트하면 다른 패키지들에 영향을 미칠 수 있습니다.

Dependency Hell은 소프트웨어의 유지 관리 및 업데이트 과정에서 예측할 수 없는 문제를 야기할 수 있습니다. 이로 인해 버그를 수정하거나 새로운 기능을 추가하는 것이 어려워지며, 개발자들은 시간을 해결책을 찾는 데 쏟아야 할 수 있습니다.

이를 해결하기 위해 npm v2에서는 어떤 방식을 사용했을까요? 바로 아래에서 확인할 수 있습니다.
아래는 npm v2에서 해당 package 들의 의존성 관계를 해석하여 생성된 node_modules입니다.

```
project/
├─ node_modules/
│   ├─ A/
│   │   ├─ node_modules/
│   │   │   └─ B@v1.0/
│   │   └─ package.json (A's dependencies: B@v1.0)
│   ├─ C/
│   │   ├─ node_modules/
│   │   │   └─ B@v2.0/
│   │   └─ package.json (C's dependencies: B@v2.0)
└─ package.json (Project's dependencies: A, C)
```

직관적으로 확인할 수 있듯, npm v2에서는 어려운 방식보단 직관적이고 쉬운 방법을 채택했습니다. 의존성관계를 해당 packages들에게 모두 남겨둔 것이지요.

위와 같은 폴더 구조 형태로 표현할 수 있으며, 아래는 [npm.github.io](https://npm.github.io/how-npm-works-docs/npm2/how-npm2-works.html)
에서 인용한 도식도 입니다.

![image](https://github.com/adultlee/study-package-manager/assets/77886826/c067950a-3487-4702-a295-b8ed7f394927)

위의 방식에서 확인할 수 있듯이, Dependency Hell을 피하기 위해서, 각각의 모든 package 들에 대해서 모든 의존성을 명시하는 것을 확인할 수 있습니다.

## npm v2 한계

npm v2에는 여러가지 한계가 있습니다.

1. npm v2에서는 의존성을 중첩하여 설치하는 방식을 사용했습니다. 이로 인해 여러 모듈이 동일한 패키지의 다른 버전을 중복으로 설치하는 경우가 발생할 수 있습니다. 이로 인해 디스크 공간 낭비와 의존성 관리의 복잡성이 증가할 수 있습니다.
2. 중첩된 의존성으로 인해 모듈의 크기가 커질 수 있습니다. 여러 모듈이 같은 패키지의 여러 버전을 중복 설치하면 디스크 공간을 비효율적으로 사용하게 됩니다.
3. npm v2의 의존성 관리 방식은 의존성을 중첩 설치하므로 설치 속도가 상대적으로 느릴 수 있습니다. 모든 의존성을 중첩적으로 설치하는 과정이 더 많은 시간을 필요로 합니다.

그렇다면 npm v2의 이러한 단점들이 npm v3에서는 어떻게 해결 되었고, 하지만 아직 가지고 있는 한계지점은 어떤 것이 있을까요?

## NPM v3

npm v3는 2015년 5월의 beta 버전 릴리즈 이후 4달뒤 9월에 정식 버전으로 [배포](https://blog.npmjs.org/post/129378362260/npm-weekly-29-npm-3-out-of-beta-nick-out-of-the)되었습니다.

npm v3는 npm v2와는 다른 방식으로 이러한 의존성 문제를 해결하였습니다. 대표적인 키워드는 바로, "Flat" 입니다.

### Flat...?

![image](https://github.com/adultlee/study-package-manager/assets/77886826/89998cc3-a9a4-4107-bd26-6ed095aa2eb0)

Flat을 검색하니 **Flat earth**가 나오는 군요... 그래도 Flat에 대해서 직관적으로 알 수 있으니, 가져와 보앗습니다.

> npm v3 의 특징은 바로 "Flat"

![image](https://github.com/adultlee/study-package-manager/assets/77886826/e6e9f343-982d-45ac-8f9b-ff59e12dd40a)
위의 사진을 보면 좀 더 이해가 빠를 수 있습니다.
NPM v3에서는 package들을 "가능한" flat하게 만들려고 합니다.

여기서 이런 생각이 들 수도 있습니다. 그렇다면 v2에서 말썽이었던, Dependency Hell 문제는 어떻게 해결한거지??

![image](https://github.com/adultlee/study-package-manager/assets/77886826/091099f8-8a19-4f3b-8973-27419791a2a2)

npm v3에서는 다음과 같은 방법으로 해결했습니다. 설치 순서(npm v3에서는 중요한 개념) 에 따라, 현재는 package A 가 package C 보다 먼저 설치 되었는데요, 이때 먼저 Flat하게 top level에 B v1.0이 올라가게 됩니다.

이 또한 tree 구조로 확인해본다면 다음과 같습니다.

```
project/
└─ node_modules/
│  ├─ A/
│  │   └─ package.json (A's dependencies: B@v1.0)
│  ├─ B@v1.0/
│  └─ C/
│  │   ├─ node_modules/
│  │   │   └─ B@v2.0/
│  │   └─ package.json (C's dependencies: B@v2.0)
└─ package.json (Project's dependencies: A, B@v1.0 ,C)
```

npm v2 에서와 가장 큰 차이점은 node_modules 의 첫 depth에 B 가 등장했다는 것입니다!
이를 통해서 차후에 B@v1.0가 더 사용될때, 더이상 하위 의존성으로 추가하지 않아도 괜찮습니다.

이를 좀더 그림을 통해서 설명 드리도록 하겠습니다.

### npm v3의 중복 처리 방법

![image](https://github.com/adultlee/study-package-manager/assets/77886826/49b8c5d1-12d4-44e1-b52d-8d6d7db38c07)

예를 들어 다음과 같은 새로운 의존성을 가진 D 패키지를 우리의 서비스에 추가한다고 생각해 봅시다.

![image](https://github.com/adultlee/study-package-manager/assets/77886826/770767f6-302e-4f33-bd08-758541a3d9dc)
그렇다면 다음과 같은 도식도가 만들어집니다!
혹시 조금 이상하신가요? npm v2의 느낌이 나며 거의 유사하다고 느끼실겁니다.
맞습니다! npm v3에서는 "가능한" flat하게 만드려 하며 , 불가능한 경우 npm v2의 기능을 사용합니다. 따라서 B v2.0을 top - level로 올릴 수 없으므로, package C 와 마찬가지로 의존성을 하위에 추가합니다.

그렇다면 B v1.0을 의존성으로 가지는 패키지를 추가해보며 개선점을 확인해보도록 하겠습니다.

![image](https://github.com/adultlee/study-package-manager/assets/77886826/36b5bde7-deb6-4a9c-9539-9a42a970979c)  
package E는 B v1.0을 의존성으로 가지는 패키지 입니다. 이 또한 저희의 서비스에 추가해보도록 하죠

![image](https://github.com/adultlee/study-package-manager/assets/77886826/27886705-055e-4e93-86ae-d8733cfefe59)

B v2.0을 추가할때와는 다른 그림이 그려졌습니다. 이는 B v1.0이 이미 top level에 있기 때문에 그려진 도식도 입니다. 따라서 이 경우에도 package A 에서와 마찬가지로 하위 dependency 없이 선언됩니다.

여기서 만약 B v1.0을 B v2.0으로 update 시켜주면 어떻게 될까요?

![image](https://github.com/adultlee/study-package-manager/assets/77886826/485cbd63-5f59-4dd2-babc-623020728d0a)

해당 서비스 내에서는 B v2.0만 사용하므로 딱봐도 간소화된 package 구조를 가지게 됨을 확인할 수 있습니다.

## npm v3의 한계

어떤가요? 그래도 npm v2에서 보다 많은 개선이 이루어진것처럼 보여집니다.
무엇보다. memory를 비효율적으로 사용하던 부분이 상당부분 개선이 된것을 확인할 수 있어요.
하지만 그럼에도 여러가지 한계가 존재 하는데요, 과연 어떤게 있을까요??

### 1. 그럼에도 불구하고 아직도 크기만한 node_modules의 크기

![image](https://github.com/adultlee/study-package-manager/assets/77886826/bfcb8f24-b366-4030-b313-bcc0bda6789c)  
[트위터에서 떠도는 node_modules 유머](https://twitter.com/CapiCapitalista/status/1499629249992474626)

다른 글에서 좀 더 설명해 드리겠지만, 아직도 node_modules는 flat 하게 했음에도 여전히 무겁습니다. 다수의 상황에서 막대한 크기로 인해서 CI 등 작업에서 문제가 발생하곤 합니다. 이를 해결하기 위해선 여러가지 해결방법이 고안되었습니다.

> 심지어 node_modules를 없애기도...(yarn berry)

### 2. 비효율적인 dependency 검색

node_modules 내부에서 dependency를 찾기 위해서는 [node_modules 로딩 방법](https://nodejs.org/api/modules.html#loading-from-node_modules-folders) 을 통해서 진행됩니다.

다음 글은 node_modules 내부에서 dependency를 찾기 위한 방법을 설명하고 있습니다.
간단하게 요약을 하자면

> 1. 현재 위치의 node_modules에서 dependency를 찾는다.
> 2. 만약 못찾았다면... 상위 폴더의 node_modules를 조사하며, 이 과정을 찾을때 까지 "반복" 합니다.

다음과 같은 dependency 검색 방법으로 인해서 복잡한 node_modules 구조를 가지는 npm의 경우 굉장히 긴 시간을 소요할 수 있습니다.

하지만 오히려 에서 발생 할 수 있는 문제는 npm v2에서는 거의 발생하지 않을 수 있습니다. 종속성을 가지는 모든 패키지들에 대해서 현재 패키지가 "모두" 가지고 있기 때문입니다. 비효율적인 디스크 활용방법이 이 경우엔 오히려 유리해진 경우 입니다.

### 3. phantom dependency 출현

![image](https://github.com/adultlee/study-package-manager/assets/77886826/485cbd63-5f59-4dd2-babc-623020728d0a)

이 도식도는 위에서 npm v3의 효율성을 설명드리며 말씀드렸던 도식도 입니다.
하지만 여기서 설명드릴 요소가 하나더 있습니다. 바로 "유령 의존성"을 의미합니다. 저는 App을 운영하면서 B v2.0을 설치한적이 없지만 마치 설치한것 처럼 사용할 수 있습니다. 이는 바로 flat 하게 옮기는 과정에서 발생한 phantom dependency라고 부르는데요, 이를 통해서 코드의 복잡성과 모호함을 늘릴 수 있습니다.

### 4. 상황과 환경마다 다른 node_modules의 구조

![image](https://github.com/adultlee/study-package-manager/assets/77886826/091099f8-8a19-4f3b-8973-27419791a2a2)
해당 도식도는 처음 npm v2에서 npm v3로 바뀌면서 생기게 된 flat 의 특징에 대해 설명하면서 추가한 도식도 입니다.
해당 도식도를 설명하기 바로 직전에서는 package A를 먼저 설치 후 package C를 설치 하였는데, 만약 이게 아니라 package C를 먼저 설치했다면, flat하게 만드려하는 npm v3에 따라 top-level에는 **B v1.0이 아닌 B v2.0**이 오게 됩니다.

# Reference

- [npm.github.io](https://npm.github.io/how-npm-works-docs/npm2/how-npm2-works.html)
