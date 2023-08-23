# About node_modules Structure in yarn

![image](https://github.com/adultlee/study-package-manager/assets/77886826/54d08b88-9296-4543-9137-1dc194206074)

yarn 은 2016년 이후로 페이스북, 구글 , Exponent 와 같은 회사들의 협력으로 인해서 개발된 새로운 package manager 입니다.
yarn을 사용하면 엔지니어들은 여전히 npm 레지스트리에 접근할 수 있지만, 패키지를 더 빨리 설치하고 여러 기계에서 일관되게 종속성을 관리하거나 보안이 유지되는 오프라인 환경에서도 사용할 수 있습니다.

## before yarn

패키지 매니저가 없던 시절에는 JavaScript 엔지니어들이 프로젝트에 직접 저장되는 소수의 종속성에 의존하거나 CDN에서 제공되는 것이 일반적이었습니다. 첫 번째 주요 JavaScript 패키지 매니저인 npm은 Node.js가 소개된 직후에 만들어졌으며, 금세 세계에서 가장 인기 있는 패키지 매니저 중 하나가 되었습니다. 새로운 오픈 소스 프로젝트가 수천 개 생성되었고 엔지니어들은 이전보다 더 많은 코드를 공유하게 되었습니다.

그러나 npm에도 여전히 여러가지 문제를 포함하고 있었습니다.

다른 기기 및 사용자 간에 종속성을 설치하는 일관성 문제, 종속성을 가져오는 데 걸리는 시간, 일부 종속성에서 코드를 자동으로 실행하는 npm 클라이언트의 실행 방식과 관련된 보안 문제 등이 발생했습니다. 이러한 문제를 해결하려고 노력했지만, 종종 새로운 문제가 발생하는 결과를 가져왔습니다.

대표적인 문제는 다음과 같은 문제들이 있었습니다.

1. 다른 기기 및 사용자 간의 종속성, 즉 non-deterministically(비결정적 - node_modules가 다를 수 있음) 현상
2. install 하는 과정이 순차적으로 이루어져 오랜 시간이 걸림

그렇다면 Yarn에서는 어떻게 문제를 해결했을까요?

## Solution by Yarn

yarn 은 npm v3의 node_modules와 같이 flat한 구조를 사용합니다. 하지만 다른점이 추가되었는데요.
yarn 에서는 non-deterministically(비결정적) 현상을 해결하기 위해, lock 파일을 제안했습니다.
이는 설치 순서와 환경, 사용자 간에 발생하는 node_modules 구조의 차이가 발생하는 문제를 해결하기 위해 도입되었으며,
이 lock 파일 내부에는 패키지의 버전과 의존성 정보가 정확하게 기록된 lock 파일을 사용하여 설치 과정을 예측 가능하게 만듭니다. 이로 인해 의존성 충돌을 피하고 설치를 빠르게 수행할 수 있도록 하였습니다. (물론 npm 진영에서도 마찬가지로 npm v5부터는 package-lock.json 이 도입되었습니다.)

즉, 동일한 lock 파일을 가지고 있는 경우 완벽히 동일한 node_modules를 구현할 수 있습니다.

또한 yarn에서는 다음의 방법을 통해 빠른 install을 구현하려 노력했습니다.

1. 병렬 설치: 패키지 매니저는 여러 개의 의존성을 동시에 설치하여 시간을 단축시킵니다. 이는 여러 개의 패키지를 한 번에 다운로드하고 설치함으로써 전체 설치 과정을 빠르게 완료할 수 있도록 합니다.

2. 캐싱: 이미 설치한 패키지는 로컬 캐시에 저장되어 다음에 같은 패키지가 필요할 때 재다운로드하지 않도록 합니다. 캐싱은 중복 다운로드를 방지하며, 패키지를 더 빠르게 설치하는 데 도움이 됩니다.

3. 빌드 캐싱: 패키지 설치 중에 필요한 빌드 과정도 캐싱하여 이전에 빌드한 결과를 재사용합니다. 이는 빌드 과정이 더욱 빠르게 이뤄질 수 있도록 합니다.
4. 보안 : yarn.lock을 먼저 확인한 후, package.json 과 비교후 문제점이 발생한다면, 바로 종료합니다. 이는 npm에서 발생했던 보안 문제를 해결하기 위한 방법이었습니다.

   결과적으로 Yarn은 다음의 구조를 가집니다.

```
├── .yarn/   - 1
│   ├── cache/   - 2
│   └── releases/   - 3
│       └── yarn-3.1.1.cjs   - 4
├── node_modules/   - 5
├── .yarnrc.yml   - 6
├── package.json   - 7
└── yarn.lock   - 8
```

1.  .yarn/: Yarn 관련 설정 및 캐시 디렉토리가 들어 있는 폴더입니다.
2.  cache/: Yarn이 내려받은 패키지와 관련된 캐시 파일이 저장되는 디렉토리입니다.
3.  releases/: Yarn의 다양한 버전과 관련된 파일이 저장되는 디렉토리입니다.
4.  yarn-3.1.1.cjs: Yarn 버전 3.1.1의 실행 파일(cjs 형식)입니다.
5.  node_modules/: 프로젝트에서 사용하는 모든 패키지의 실제 코드가 들어 있는 디렉토리입니다.
6.  .yarnrc.yml: Yarn의 설정 파일인 .yarnrc.yml 파일로, 프로젝트의 Yarn 구성 옵션을 지정할 수 있습니다.
7.  package.json: 프로젝트의 메타 정보와 종속성을 정의하는 파일로, 패키지 버전 및 스크립트 등을 설정할 수 있습니다.
8.  yarn.lock: 패키지 의존성의 정확한 버전을 보장하기 위한 Yarn 락 파일로, 패키지 버전 및 의존성 트리가 고정되어 있습니다.

## yarn vs npm

![image](https://github.com/adultlee/study-package-manager/assets/77886826/20541db3-1620-4c80-986d-a4cbca672b2f)
사진에서확인할 수 있듯, 해당 레포에 gnomon 라이브러리를 사용하여, 각 패키지의 종료시점을 확인한 결과 npm 보다 yarn이 더 빠른 속도를 보임을 확인했습니다.

> npm : 37초
> yarn : 26초
> 상세한 제한 조건은 해당 repo에서 확인하실 수 있습니다.

## 여담

물론 npm에서도 이런 yarn의 노력에 뒤지지 않고 npm v5 부터는 거의 동일한 기능을 수행할 수 있도록 구현되었습니다.
하지만 아직도 yarn과의 유의미한 속도와 성능차이가 있습니다.

또한 yarn 이 v2가 출시되고, legacy로 분류되어 개발이 종료되었다는 점을 인지할 필요가 있습니다!

## Yarn의 한계

yarn은 npm v3가 가지고 있던 여러가지 문제(속도, 보안) 등등을 획기적으로 해결했습니다만, 아직 node_modules를 구현함에 있어서 막대한 시간과 자원을 사용하고 있다는 한계가 있습니다. 이를 해결하기 위해서 다른 package manager 들이 필요성이 대두되었습니다.

# Reference

- [npm.github.io](https://npm.github.io/how-npm-works-docs/npm2/how-npm2-works.html)
- [npm docs](https://blog.npmjs.org/post/129378362260/npm-weekly-29-npm-3-out-of-beta-nick-out-of-the)
- [직방 기술블로그](https://medium.com/zigbang/%ED%8C%A8%ED%82%A4%EC%A7%80-%EB%A7%A4%EB%8B%88%EC%A0%80-%EA%B7%B8%EA%B2%83%EC%9D%B4-%EA%B6%81%EA%B8%88%ED%95%98%EB%8B%A4-5bacc65fb05d)
- [토스 기술블로그](https://toss.tech/article/node-modules-and-yarn-berry)
- [Yarn 공식 블로그](https://engineering.fb.com/2016/10/11/web/yarn-a-new-package-manager-for-javascript/)
- [npm vs yarn vs pnpm](https://yceffort.kr/2022/05/npm-vs-yarn-vs-pnpm#%EB%A7%8E%EC%9D%80-%ED%98%81%EB%AA%85%EC%9D%84-%EA%B0%80%EC%A0%B8%EC%98%A8-yarn-classic)
