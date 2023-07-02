### 참고 자료

[최적화 웹 성능 최적화 방법 5분 완성](https://velog.io/@hsecode/최적화-웹-성능-최적화-방법-5분-완성)

[최적화 체크리스트](https://github.com/parksb/Front-End-Performance-Checklist)

### 성능 최적화 도구

- [lighthouse](https://developer.chrome.com/docs/lighthouse/overview/)
- [pagespeed insight](https://pagespeed.web.dev/?utm_source=psi&utm_medium=redirect&hl=ko)
- [webpage test](http://catchpoint.com/webpagetest)

### 성능 최적화 체크리스트

- [ ] HTML / CSS / JS 압축하기 (주석, 공백, 줄바꿈 제거)
- [ ] CSS는 반드시 `<head>`에 JS는 `<body>` 끝에 or defer나 async 사용
- [ ] `iframe` 최소화
- [ ] CSS 파일은 합치기
- [ ] CSS 파일은 `non-blocking` 되게끔
  ```jsx
  <link rel="preload" href="global.min.css" as="style" onload="this.rel='stylesheet'">
  <noscript><link rel="stylesheet" href="global.min.css"></noscript>
  ```
- [ ] 사용하지 않는 CSS 선택자는 반드시 지워주기
- [ ] 리플로우 / 리페인트 최소화하기
  - 브라우저 동작 원리 알아보기
  - **리플로우:**
    레이아웃의 넓이, 높이, 위치 등에 영향을 주는 css 속성을 변경할 경우 'Layout'부터 다시 그려지게 되는 과정
    > JS, CSS 파싱 → 렌더 트리 구축 → 레이아웃 → 리페인트 → 레이어 업데이트 → 합성
    → 리플로우를 발생시키는 속성:
    ```jsx
    position / width / height / margin / padding / display / top / left / right / bottom /
    box-sizing / border-color / text-align / border / border-width /
    font-family / float / font-size / font-weight / line-height / vertical-align /
    white-space / word-wrap / text-overflow / text-shadow ...
    ```
  - **리페인트:**
    레이아웃에 영향을 주지 않는 속성을 변경하면 레이아웃을 건너뛰고 페인트 작업부터 다시 수행하는 과정
    > JS, CSS 파싱 → 렌더 트리 구축 → 리페인트 → 레이어 업데이트 → 합성
    → 리페인트를 발생시키는 속성:
    ```jsx
    color / border-style / visibility / background / background-color /
    background-image / background-position / background-repeat / background-size /
    text-decoration / outline / outline-style / outline-color / outline-width /
    border-radius / box-shadow ...
    ```
  - **관련 세부 체크리스트:**
    - [ ] 최대한 DOM 구조 상 말단 노드에만 클래스를 사용
      → 리플로우의 영향을 최소화하여 수행 비용을 줄이기 (리플로우는 자식/조상/부모까지 조사)
    - [ ] **인라인 스타일 자제**
      → 리플로우 수차례 발생, 클래스 사용 지향
    - [ ] 애니메이션의 position은 `absolute`, `fixed` 지향
      → 주변 레이아웃 영향 최소화
    - [ ] JS에서 CSS 사용 시, 가급적 한 번에 처리
      → **👿 나쁜 코드**
      ```jsx
      toChange.style.background = "#333";
      toChange.style.color = "#fff";
      toChange.style.border = "1px solid #ccc";
      ```
      → **😇 좋은 코드**
      ```jsx
      /* CSS */
      #elem { border:1px solid #000; color:#000; background:#ddd; }
      .highlight { border-color:#00f; color:#fff; background:#333; }
      /* js */
      document.getElementById('elem').className = 'highlight';
      ```
    - [ ] position `relative`는 상대적으로 큰 비용이 들기 때문에 사용 시 주의
      → 자세한 내용:
      [](https://lists.w3.org/Archives/Public/public-html-ig-ko/2011Sep/att-0031/Reflow_____________________________Tip.pdf)
- [ ] 사용하지 않는 CSS 파일 제거
  - 크롬의 lighthouse에서 확인할 수 있음
- [ ] 인라인 스타일 **지양**
- [ ] picture 태그 이용해서 이미지 최적화
  ```jsx
  <!-- 브라우저가 avif를 지원하면 avif를 사용하고,
       그렇지 않은 경우 webp,
      둘 다 지원하지 않을 경우 jpg 이미지를 사용한다. -->
  <picture>
      <source srcset="aaa.avif" type="image/avif">
      <source srcset="aaa.webp" type="image/webp">
      <img src="aaa.jpg" alt>
  </picture>
  ```
  ```jsx
  <picture>
      <source srcset="mob.webp" media="(max-width: 760px)"> <!-- 브라우저의 넓이가 760px 이하일때 mob.webp 이미지 출력-->
      <img src="pc.webp" alt>
  </picture>
  ```
- [ ] 이미지 로딩 지연 사용
  ```jsx
  <img src="item.jpg" loading="lazy" alt>
  ```
- [ ] 웹팩을 통해서 번들링
- [ ] Gzip 사용해서 텍스트 기반 리소스 압축
- [ ] CDN 사용
- [ ] 캐싱 사용
  [참고 링크](https://hahahoho5915.tistory.com/33)
