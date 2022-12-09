# monorepo-practice.js

모노레포란 잘 정의된 관계를 가진, 여러개의 독립적인 프로젝트들이 있는 하나의 레포지토리

- 멀티레포 : Project A와 비슷한 Project B를 생성
- 새로운 Project B 레포지토리를 추가 ci/cd, lint, test, ts 등등 설정을 새로 해준다.
- Project A에서 재사용하고 싶은 코드가 있는 경우 이를 위한 Library A 레포지토리를 새로 만들어준다.
- 반복..

#### 멀티레포의 문제점

- 새 프로젝트 생성 비용이 큼
- 프로젝트간 코드 공유가 어려움
- 같은 이슈를 수정하기 위해 각각의 레포지토리에 커밋이 필요
- 히스토리 관리 어려움
- 제각각인 툴링으로 개발자 경험이 일관적이지 않음

#### 모노레포가 해결해 주는 것

- 새 프로젝트 생성 비용이 작음
- 프로젝트간 코드 공유가 쉬움
- Atomic commits
- 히스토리 관리 쉬움
- 공통된 툴링으로 일관적인 개발자 경험

#### 모던 모노레포 툴들에서 제공하는 기능들

```
속도
```

- Local computation caching
- Local task orchestration
- Distributed computation caching
- Detecting affected projects / packages

```
관리
```

- Source code sharing
- Code generation
- Project constraints and visibility
- Toss Frontend Library

#### 기술적 요구사항

```
의존성 관리
```

- 유령 의존성 (Phantom Dependency) : 명시되어 있지 않은 의존성. 많은 문제를 일으킴
- yarn berry + PnP

```
no more hoisting
.pnp.cjs를 통한 엄격한 의존성 관리
Zero install
```

- 빠른 의존성 검색

```
Peer dependency : 상속된 의존성
Peer dependency는 전파된다.
Peer dependency 오류는 잠재적 런타임 에러이다.
꼭 싱글턴으로 존재해야 하는 패키지일 때만 Peer Dependency로 명시해주자
```

#### 버전 관리

- Semver : Semantic Versioning
- Major / Minor / Patches
- 라이브러리에서는 버전을 더 잘 지켜야 한다.
- lerna version

#### 코드 품질 관리

- RFC (Request For Comments) : 아이디어 단계에서 PR 전에 기여하고 싶은 부분을 올림
- PR
  코드 리뷰

```
CI가 동작하면서 코드 검증
peer dependency error check
unused export check
pre build check
canary version check
```

#### 문서화

- 어떻게 사용하는지 문서 중요
- JSDoc, Docusaurus 로 자동으로 문서화
- slash → 획을 긋다
- https://github.com/toss/slash → 토스 프론트엔드 챕터에서 사용하는 모노레포

#### 순서

- 1.yarn berry 버전 변경

```
// yarn 버전 확인
yarn -v

// yarn 버전 변경
yarn set version berry

// yarn 버전 확인
yarn -v
```

- 2.yarn workspace 패키지 만들기

```
// packages 디렉토리 만들기 / 루트 초기화
yarn init -w
```

- 3.root pacakge.json파일 수정

```
./package.json
apps/* 폴더추가

{
  "name": "yarn-berry-workspace-test",
  "packageManager": "yarn@3.3.0",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*"
  ]
}
```

- 4.apps 폴더에 create next-app 프로젝트 추가

```
cd apps
yarn create next-app
```

- 5.typescript error 발생

```
root 폴더에서
yarn add -D typescript
yarn dlx @yarnpkg/sdks vscode
(저는 이상하게 향후 packages의 lib 의존성을 추가해야 에러가 사라졌음)
```

- 6.yarn PnP 사용을 위한 vscode extension 설치

```
.vscode/extensions.json
{
  "recommendations": ["arcanis.vscode-zipfs"]
}
```

- 7.pacakges/lib 공통 패키지를 만들어보기

```
packages/lib 폴더 생성하고, 아래 스크립트를 실행한다.

// lib 폴더 이동
cd packages/lib

// pacakge.json 파일 생성
yarn init

아래와 같이packages/lib/package.json 파일 수정한다.

{
  "name": "@mono/lib",
  "version": "1.0.0",
  "private": true,
  "main": "./src/index.ts",
  "dependencies": {
    "typescript": "^4.9.3"
  }
}

packages/lib/tsconfig.json 파일 생성 후 설정값 넣기

packages/lib/src/index.ts 생성후
간단한 공용 컴포넌트 만들기
```

- 8.apps/mono pacakges/lib 의존해 보기

```
apps/mono 의존성 주입

// root로 이동한다.
cd ../../

// root 실행한다.
yarn workspace @mono/web add @mono/lib
apps/mono/package.json에 의존성이 추가된 것을 확인 합니다.
```

- 9.모노레포 구동

```
yarn workspace @wanted/web run dev
```
