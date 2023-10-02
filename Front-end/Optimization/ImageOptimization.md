## 이미지 성능 최적화

- **width / height 값 정확히 지정해주어서 리플로우를 방지하자!**
- **여러 버전의 이미지를 제공하자**
    
    ```html
    <img
      srcset="images/heropy_small.png 400w,
              images/heropy_medium.png 700w,
              images/heropy_large.png 1000w"sizes="(max-width: 500px) 444px,
             (max-width: 800px) 777px,
             1222px"src="images/heropy.png"alt="HEROPY" />
    ```
    
    ```jsx
    <picture><source media="(max-width: 760px)" srcSet={mobileUrl} /><source media="(min-width: 761px)" srcSet={desktopUrl} /></picture>
    ```
    
- **lazyloading 활용**
    - 지금 당장 로드 할 필요 없는 이미지는 필요할 때 로드 하면 됨!
    - 예를 들어서, 20개짜리 carousel을 구현할 때 이미지 20개를 처음에 모두 받아올 필요는 없다
        - 그렇게 된다면, 페이지 로드 초기에 블로킹이 발생함
        - 그래서 현재 보여지는 것들만 지금 로드 하고 다른 것들은 나중에 보여질 때 로드 하는게 필요!
    - 사용 방법
        - 외부 라이브러리 사용 (lazy size)
        - browser의 loading=lazy 옵션 이용
        - **intersection observer**를 이용하는 방법은 data-src, src를 이용하여 초기에는 스켈레톤 이미지, 뷰포트에 도달할시 data-src에 있는 것을 src로 옮겨 화면에 보여주도록 하는데 체감상 loading='lazy' 보다 자연스럽지 못합니다.
            
            역시 intersection observer를 지원하지 않는 경우가 있기 때문에 polyfill이 필요합니다.
            
- **fetchPriority**
    - 초반에 빨리 보여져야 하는 이미지에 사용
        
        `<img fetchPriority="high" />`
        
    - ts에 없어서 d.ts를 통해서 추가해줘야함
        
        ```tsx
        import { AriaAttributes, DOMAttributes } from 'react';
        
        declare module 'react' {
          interface HTMLAttributes<T> extends AriaAttributes, DOMAttributes<T> {
            fetchPriority?: 'high' | 'low' | 'auto';
          }
        }
        ```
        
- **lqip 활용**
    - lqip: low quality image placeholder
    - 저용량의 이미지를 썸네일로 사용하고 → 로드 다 되면 이미지로 대체 시키기!
    - 흔히 보는 블러 처리된 이미지!
