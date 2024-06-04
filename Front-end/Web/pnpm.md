## Intro
최근에 동아리에서 운영중인 프로젝트에 모노레포를 도입하고 있다. (관련된 내용은 다른 게시물에서 본격적으로 다루어보겠다!)

해당 프로젝트에서 pnpm의 workspace를 이용하기 위해서 패키지 매니저를 pnpm을 채택했다. 

그래서 pnpm에 대한 내용을 블로그에 정리해보고자 한다!

## 왜 만들어졌는가?
pnpm의 탄생 배경을 알아보기 위해서는 npm의 문제점을 먼저 알아봐야한다.
npm은 **v2->v3로 업데이트** 되면서 큰 변화를 겪었는데, 이 때의 변화를 먼저 알아보자.

### npm2 vs npm3
![npm v2 and npm v3](https://velog.velcdn.com/images/intersoom/post/56944734-be31-4bf4-9509-3e06a758c717/image.png)

### 📍 npm v2

**npm v2**에서는 패키지를 설치하면, 모듈 각각이 각자의 `node_modules` 파일을 가진다.

**장점 :** 각자 다른 의존성 버전을 가지는 모듈들이 각자 자신의 하위 디렉토리에 있는 모듈을 참고하기 때문에 모듈이 가진 의존성들의 충돌을 신경 쓰지 않아도 된다.

```
node_modules/
├── a/
│   └── node_modules/
│       └── l/ # v1.00
└── b/
    └── node_modules/
        └── l/ # v2.00
```
**단점 :** 이는 같은 의존성 버전을 가지는 경우에도 구조가 똑같이 구성되기 때문에, 불필요한 파일들이 설치가 된다.


```
node_modules/
├── a/
│   └── node_modules/
│       └── l/ # v1.00
└── b/
│   └── node_modules/
│       └── l/ # v1.00
└── c/
    └── node_modules/
        └── l/ # v1.00
.
.
.
```
위의 예시처럼 같은 모듈이 10번 혹은 그 이상 설치될 가능성이 존재한다.

### 📍 npm v3

그래서 위의 _npm v2의 단점을 해결하고 npm v2의 장점을 유지하는 방식_ 으로 **npm v3**가 고안되었다.

**npm v3**에서는 중복되는 모듈을 depth가 1이도록 **호이스팅 해서** 폴더 구조가 **flat**하게 만들어준다. 
그리고 버전 충돌이 발생할 수 있는 경우에는 호이스팅 하지 않고 **nested**하게 유지한다.

> **node.js의 require() 우선 순위 특징**
> 
> _1) 자신 하위의 node_modules_
> _2) 상위 폴더_

예시를 보면 더 쉽게 이해가 가능하다 💡

**_v2:_**
```
node_modules/
├── a/
│   └── node_modules/
│       └── l/ # v1.00
└── b/
│   └── node_modules/
│       └── l/ # v1.00
└── c/
    └── node_modules/
        └── l/ # v1.00
.
.
.
```
**_v3:_**

```
node_modules/
├── a/
├── b/
├── c/
└── l/ # v1.0.0
```
모두 같은 버전의 l을 사용하고 있으니 flat하게 하나의 v1.0.0의 l 모듈 1개만 루트에 설치하면 된다.

a, b, c 모두가 하위에 node_modules 폴더를 갖지 않으니까 상위에 있는 l 모듈을 참조할 것이다.

그렇다면, 만약 a와 b가 각각 _다른 버전의 l_ 에 의존성을 보유한다면 어떻게 될까?

```
node_modules/
├── a/
├── b/
│   └── node_modules/
│       └── l/  # v2.0.0
└── l/   # v1.0.0 
```

1. a를 설치하면서, 루트에 v1.0.0의 l을 설치 
2. b를 설치하면서 루트에 이미 l v1.0.0이 설치 되어있는 것을 확인
3. b의 하위에 l v2.0.0 설치

이렇게 되면, require()의 우선순위에 따라서 b는 자신 하위의 l(v2.0.0)을 참조하고, a는 하위에 node_modules를 가지지 않으니 상위의 l(v1.0.0)을 참조하게 된다.

> **만약 추가로 설치한 c 모듈이 l v2.0.0을 참조한다면?**
>
> c 하위의 v2.0.0의 l을 추가적으로 설치할 것이다.
> -> 이를 통해서 _node_modules의 구조는 설치 순서의 영향_ 을 받는다는 사실을 알 수 있다.

### 정리
간단하게 정리하자면, 
- **npm v2 :** 각 모듈의 의존성을 각자의 하위 디렉토리에 설치한다
- **npm v3 :** 중복되는 모듈들을 호이스팅하여 root에 설치한. 의존하는 모듈의 버전이 다른 경우에 한해서, 이후에 설치한 모듈의 하위 디렉토리에 루트에 설치된 모듈과 다른 버전을 설치한다.

### npm의 문제점
_이제부터는 npm v3를 기준으로 작성하겠다._

npm v2의 문제를 개선한 npm v3에는 그러면 어떠한 문제점들이 존재했는지 알아보자!

#### 📍 비효율적인 의존성 탐색
npm은 node_modules 폴더를 이용해서 의존성을 탐색한다. 이렇게 관리하면, 상당히 비효율적인 의존성 탐색이 이루어진다.

하나의 폴더에서 특정 패키지를 불러오는 상황을 예를 들어보면,
패키지를 찾기 위해서 계속해서 상위 디렉토리의 node_modules를 탐색한다.

이해를 돕기 위해서 [토스에서 제공한 예시](https://toss.tech/article/node-modules-and-yarn-berry)를 첨부하겠다.

`/Users/toss/dev/toss-frontend-libraries` 폴더에서 `require()` 문을 이용하여 react 패키지를 불러오는 상황을 가정해보자.

> **`require.resolve.paths()` 함수란?**
>
> 라이브러리를 찾기 위해 순회하는 디렉토리의 목록을 확인할 수 있게 해주는 함수

```
$ node
Welcome to Node.js v12.16.3.
Type ".help" for more information.
> require.resolve.paths('react')
[
  '/Users/toss/dev/toss-frontend-libraries/repl/node_modules',
  '/Users/toss/dev/toss-frontend-libraries/node_modules',
  '/Users/toss/node_modules',
  '/Users/node_modules',
  '/node_modules',
  '/Users/toss/.node_modules',
  '/Users/toss/.node_libraries',
  '/Users/toss/.nvm/versions/node/v12.16.3/lib/node',
  '/Users/toss/.node_modules',
  '/Users/toss/.node_libraries',
  '/Users/toss/.nvm/versions/node/v12.16.3/lib/node'
]
```

#### 📍 용량 낭비
npm에서는 프로젝트마다 각자의 dependecy가 각각 프로젝트에서 설치 되기 때문에 불필요한 용량들이 낭비된다.

예를 들어, 프로젝트가 3개에서 같은 dependecy를 가지는 경우, 같은 파일이 3번 설치되는 것이다. 
그렇기 때문에, 각각의 프로젝트의 용량도 너무 커질 뿐더러 프로젝트를 몇 개만 생성하게 되면 디스크의 용량이 포화 상태가 되어버린다.

#### 📍 유령 의존성
개인적으로 해당 문제가 가장 큰 문제라고 생각한다.

본인이 속해 있는 동아리에서도 유령 의존성 문제 때문에 패키지 매니저를 yarn에서 pnpm으로 변경한 사례가 있다.

그래서 유령의존성이 무엇인가?
앞서, npm v3부터는 호이스팅을 통해서 패키지의 중복 설치를 막는다고 언급했다.
이에 따라서, 원래는 루트에서는 require()이 불가능했던 패키지들까지 불러올 수 있게 된 것이다.

예를 들어서,
원래는 다음과 같은 의존성 트리를 가지는데

```
node_modules/
├── a/
├── b/
└── c/
	└── l/ 
```
호이스팅으로 인해 다음과 같은 트리를 가지게 되었다고 해보자.
```
node_modules/
├── a/
├── b/
├── c/
└── l/ 
```
그렇다면, 본래는 참조할 수 없었던 l이 root에서 참조가 가능해진다.

이는 package.json에서 명시하지 않은 패키지들을 유령처럼 조용히 사용할 수 있게 하고, 또한 조용히 없앨 수도 있게 된다. 이는 의존성 관리 시스템에 혼란을 야기할 수 있다.

### npm vs pnpm
그렇다면, pnpm은 npm과 뭐가 그렇게 다른가?
지금부터 알아보기.. 전에 심볼릭 링크와 하드 링크에 대해서 간단하게만 알아보고 넘어가자 !

#### 📍 심볼릭 링크 vs 하드 링크

![](https://velog.velcdn.com/images/intersoom/post/5b088ee8-8897-47dc-9126-934099a0db76/image.png)


- **하드 링크**
	원본 파일과 동일한 inode를 가리키는 링크이다.
    그래서 원본이 삭제 되더라도, 원본 파일 내용에 접근이 가능하다.
    ~~Inode, 데이터 블록 같은 개념들은 차차 CS 내용을 정리하면서 더 공부해보겠다.. 🥲~~

    쉽게 말하면, 데이터가 존재하는 **주소값을 원본 파일과 동일하게 가지고 있는 것**이다. 그렇기 때문에 원본 파일이 삭제 되더라도 데이터가 존재하는 메모리에 직접 접근이 가능하게 되는 것이다.

- **심볼릭 링크**
	심볼릭 링크는 원본 파일과는 또 다른 inode를 생성하여서 이를 바라보게 된다. 이 때 생성된 inode가 **원본 파일을 바라보는 포인터**를 바라보게 된다.
    그렇기 때문에 원본 파일이 삭제 되면 하드 링크와는 다르게, 파일의 내용에 접근할 수 없게 된다. 

이제 진짜 pnpm에 대해서 이야기 해보겠다.

#### pnpm store

pnpm은 먼저, 아래와 같이 `pnpm store`를 디바이스 전역에 생성하고 패키지들을 이 곳에 설치한다.

![](https://velog.velcdn.com/images/intersoom/post/7b65ba87-f9e8-4645-b0d9-1248c1d862f8/image.png)

디렉토리를 타고 들어가보면 아래와 같이 해시 값의 형태로 패키지들이 저장되어 있는 모습을 확인할 수 있다.

![](https://velog.velcdn.com/images/intersoom/post/1c8338d2-d865-47cc-883e-ff157724ce8a/image.png)

이렇게 pnpm store라는 전역의 공간에 패키지들의 원본 파일이 존재한다.

> **💡 용량 문제를 해결하였다**
> 같은 패키지를 전역에 설치 해두고 이를 참조해서 프로젝트에 이용하기 때문에 엄청난 용량 절감이 가능해진다!

#### 하드링크 : node_modules/.pnpm

프로젝트를 생성하면 전역 pnpm store의 패키지들에 대한 하드 링크가 걸린 패키지 링크들이 존재한다.

![](https://velog.velcdn.com/images/intersoom/post/03cf9126-0b2b-4cc4-9aa9-2da617de6682/image.png)

#### 심볼릭 링크 : node_modules/

node_modules 아래에는 해당 프로젝트의 의존성 패키지들이 쭉 설치가 되어 있는데, 이는 `node_modules/.pnpm`에 존재하는 파일들에 대한 심볼릭 링크이다. 

![](blob:https://velog.io/46a73ad1-97f1-4914-9153-695f5cc28446)

아래와 같이 명령어를 입력해봤을 때, 심볼릭 링크의 형태를 조금 더 직관적으로 확인 가능하다!

![](https://velog.velcdn.com/images/intersoom/post/573f1802-0c77-465c-a0fc-37ebd9a3f645/image.png)

> **💡 속도 문제를 해결하였다**
> _npm_ 에서는 비효율적인 의존성 탐색 문제로 설치 속도가 굉장히 저하되는 문제가 존재했다.
>
> _pnpm_ 에서는 패키지 별로 설치하는 순서를 고려할 필요조차 없어졌다.
> 그렇기 때문에, 병렬적으로 필요한 패키지들을 전역에 설치한다. 
> 
> 그리고 의존성 트리에 맞춰서 링크만 걸어주면 되게끔 simple하게 설치 과정이 이루어지기 때문에, 상당히 빠르게 이루어진다.

> **💡 유령 의존성 문제를 해결하였다**
> 특정 패키지를 pnpm store에 존재하는 하나의 패키지를 참조해서 사용하는 것이기 때문에 의존성 문제를 완벽하게 해결한다.
>
> 각자의 패키지들이 자신에게 알맞은 의존성 트리를 유지하면서도, 용량 문제, 중복 설치 등의 문제를 해결한다!
> 
> 프로젝트 내에 설치 되지 않은 패키지가 참조되는 **유령 의존성 문제** 같은 일은 발생하지 않는다 !

이렇게 npm의 간략한 역사와 pnpm의 등장 배경부터 작동 원리까지 알아보았다!
꽤나 흥미로운 내용이었고, 이 모든 것을 이해하는 과정이 굉장히 재밌었다.

**하나를 써도 제대로 알고 쓰자** 라는 생각이 더 짙어진 시간들이었다 😇

다음에는 현재 모노레포 구축 과정에서 쓰고 있는 **pnpm의 workspace**에 대해서 정리해보겠다!

### Reference
https://pnpm.io/ko/blog/2020/05/27/flat-node-modules-is-not-the-only-way
https://jeonghwan-kim.github.io/2023/10/20/pnpm#%ED%8F%B4%EB%8D%94-%EA%B5%AC%EC%A1%B0-pnpm
https://lasbe.tistory.com/200
https://blog.outsider.ne.kr/1230
https://npm.github.io/how-npm-works-docs/npm3/how-npm3-works.html
