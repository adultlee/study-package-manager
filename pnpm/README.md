# About node_modules Structure in Pnpm

![image](https://github.com/adultlee/study-package-manager/assets/77886826/fc0c96e0-d907-448f-b6a8-5a264c888921)

pnpm은 패키지 의존성 관리 도구로, npm과 비슷한 목적을 가지고 있지만 독특한 방식으로 패키지를 설치하고 관리합니다. pnpm은 "퍼포먼스"와 "디스크 공간 절약"을 중요한 가치로 내세우며, 다른 패키지 매니저들과는 조금 다른 작동 방식을 갖고 있습니다.

1. flat 한게 정답이엇을까?
2. phantom dependency 해결

## flat 한게 정답이엇을까?

npm v3 와 yarn의 node_modules 구조는 Flat 한 형태로 개발되었습니다. 이 당시에는 획기적인 방법이었습니다. npm v2 가 가지는 고질적인 복잡하고 무거운 node_modules 구조가 어느정도 개선되었기 때문이죠. 하지만 아직 산재한 여러가지 문제가 존재했습니다.
phantom dependency 가 존재하거나, 그럼에도 불구하고 아직도 무겁고 느린 node_modules 때문이었습니다.

여기서 pnpm 은 획기적인 생각을 하게 됩니다. 바로 npm v2의 답을 찾아 떠나는 선택을 한 것이죠.

![image](https://github.com/adultlee/study-package-manager/assets/77886826/b4dc34fd-2274-4cd5-b45e-483fca50175b)

> https://yceffort.kr/2022/05/npm-vs-yarn-vs-pnpm

pnpm의 node_modules 레이아웃은 심볼릭 링크를 사용하여 의존성의 중첩 구조를 생성합니다.

node_modules 안에 있는 모든 패키지의 모든 파일은 콘텐츠 주소 지정 저장소에 대한 하드 링크입니다. bar@1.0.0에 의존하는 foo@1.0.0 을 설치했다고 가정해 봅시다. pnpm은 다음과 같이 두 패키지를 모두 node_modules 에 하드 링크합니다.

```
node_modules
└── .pnpm
    ├── bar@1.0.0
    │   └── node_modules
    │       └── bar -> <store>/bar
    │           ├── index.js
    │           └── package.json
    └── foo@1.0.0
        └── node_modules
            └── foo -> <store>/foo
                ├── index.js
                └── package.json
```

이것은 node_modules의 유일한 "실제" 파일입니다. 모든 패키지가 node_modules에 하드 링크되면, 중첩된 의존성 그래프 구조를 구축하기 위해 심볼릭 링크가 생성됩니다.

눈치채셨겠지만 두 패키지 모두 node_modules 폴더 안의 하위 폴더에 연결되어 있습니다 (foo@1.0.0/node_modules/foo). 이것은 다음을 위해 필요합니다:

패키지가 자기 자신에 대한 import를 허용합니다. foo 는 require('foo/package.json') 또는 import \* as package from "foo/package.json" 를 할 수 있어야 합니다.
순환 심볼릭 링크를 피합니다. 패키지의 의존성은 의존하는 패키지와 동일한 폴더에 있습니다. Node.js의 경우, 의존성이 패키지의 node_modules 내부에 있는지 또는 상위 디렉토리의 다른 node_modules 에 이 있는지 여부로 차이를 두지 않습니다.
설치의 다음 단계는 의존성을 심볼릭 링크하는 것입니다. bar 는 foo@1.0.0/node_modules 폴더에 심볼릭 링크될 겁니다.

```
node_modules
└── .pnpm
    ├── bar@1.0.0
    │   └── node_modules
    │       └── bar -> <store>/bar
    └── foo@1.0.0
        └── node_modules
            ├── foo -> <store>/foo
            └── bar -> ../../bar@1.0.0/node_modules/bar
```

다음으로, 직접 의존성이 처리됩니다. foo 는 루트 node_modules 폴더에 심볼릭 링크됩니다. 그 이유는 foo 가 프로젝트의 의존성이기 때문입니다.

```
node_modules
├── foo -> ./.pnpm/foo@1.0.0/node_modules/foo
└── .pnpm
    ├── bar@1.0.0
    │   └── node_modules
    │       └── bar -> <store>/bar
    └── foo@1.0.0
        └── node_modules
            ├── foo -> <store>/foo
            └── bar -> ../../bar@1.0.0/node_modules/bar
```

이것은 매우 간단한 예입니다. 그러나 이 레이아웃은 의존성 수와 의존성 그래프의 깊이에 관계없이 이 구조를 유지합니다.

bar 및 foo의 의존성으로 qar@2.0.0 을 추가해 보겠습니다. 새로운 구조는 다음과 같습니다.

```
node_modules
├── foo -> ./.pnpm/foo@1.0.0/node_modules/foo
└── .pnpm
    ├── bar@1.0.0
    │   └── node_modules
    │       ├── bar -> <store>/bar
    │       └── qar -> ../../qar@2.0.0/node_modules/qar
    ├── foo@1.0.0
    │   └── node_modules
    │       ├── foo -> <store>/foo
    │       ├── bar -> ../../bar@1.0.0/node_modules/bar
    │       └── qar -> ../../qar@2.0.0/node_modules/qar
    └── qar@2.0.0
        └── node_modules
            └── qar -> <store>/qar
```

보시다시피 그래프가 더 깊어지더라도 (foo > bar > qar), 파일 시스템의 디렉토리 깊이는 여전히 동일합니다.

이 레이아웃은 언뜻 보기에는 이상해보일 수 있지만, Node의 모듈 resolution 알고리즘과 완벽하게 호환됩니다. 모듈을 확인할 때, Node는 심볼릭 링크를 무시하므로, foo@1.0.0/node_modules/foo/index.js 에서 bar 가 필요할 때 Node는 foo@1.0.0/node_modules/bar 에서 bar 를 사용하지 않습니다. 대신, bar 는 실제 위치로 확인됩니다(bar@1.0.0/node_modules/bar). 결과적으로, bar 는 bar@1.0.0/node_modules에 있는 의존성을 해결할 수도 있습니다.

# phantom dependency 해결

![image](https://github.com/adultlee/study-package-manager/assets/77886826/77c0a6f2-0bc6-40fd-aaa0-018433f1ec45)
다음과 같은 구조의 가장 큰 장점은 phantom dependency를 해결할 수 있다는 점이엇습니다.

# Reference

- [npm.github.io](https://npm.github.io/how-npm-works-docs/npm2/how-npm2-works.html)
- [npm docs](https://blog.npmjs.org/post/129378362260/npm-weekly-29-npm-3-out-of-beta-nick-out-of-the)
- [직방 기술블로그](https://medium.com/zigbang/%ED%8C%A8%ED%82%A4%EC%A7%80-%EB%A7%A4%EB%8B%88%EC%A0%80-%EA%B7%B8%EA%B2%83%EC%9D%B4-%EA%B6%81%EA%B8%88%ED%95%98%EB%8B%A4-5bacc65fb05d)
- [토스 기술블로그](https://toss.tech/article/node-modules-and-yarn-berry)
- [Yarn 공식 블로그](https://engineering.fb.com/2016/10/11/web/yarn-a-new-package-manager-for-javascript/)
- [npm vs yarn vs pnpm](https://yceffort.kr/2022/05/npm-vs-yarn-vs-pnpm#%EB%A7%8E%EC%9D%80-%ED%98%81%EB%AA%85%EC%9D%84-%EA%B0%80%EC%A0%B8%EC%98%A8-yarn-classic)
