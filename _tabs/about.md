---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---

## 경력

### [Imagoworks](https://dentbird.kr/)

_2025.02 ~ 재직중 (백엔드 개발자)_

<br/>

### [SI Analytics](https://si-analytics.ai/)

_2022.08 ~ 2025.02 (2년 7개월, 백엔드 개발자)_

#### 업무

- gRPC와 RabbitMQ로 통신하며 동작하는 **MSA 구조의 시스템에서 15개의 서버를 개발 및 유지보수**
- AI 기반 이상 탐지와 변화 탐지 기능의 **아키텍처, 프로세스, DB를 설계 및 구현**
- **서비스와 개발 환경 개선**
  - **서버 안정화** : [DB 세션 누수 해결](https://leesm0518.github.io/posts/database-session-leak/)
  - **성능 개선** : [검수 저장 성능 개선](https://leesm0518.github.io/posts/save-review-performance-tuning/), [위성 영상 삭제 성능 개선](https://leesm0518.github.io/posts/delete-scene-performance-tuning/), [위성 영상 저장 성능 개선](https://leesm0518.github.io/posts/save-scene-performance-tuning/)
  - **시스템 개선** : [위성 영상 전처리 서버 통합](https://leesm0518.github.io/posts/scene-preprocessing-servers-integration/)
  - 시스템을 납품할 고객사 증가에 따른 **새로운 브랜치 전략 도입과 CI/CD 구축**
  - **CI/CD 성능 개선**
- 온프레미스로 **k8s 환경 기반의 시스템을 구축**해 납품하거나, AWS의 **EKS를 활용해 배포 및 운영**
- 문서 공유 활성화를 위해 **백엔드 문서 및 일정 관리 페이지를 개선하고, 작업 및 기술 문서를 약 500개 작성** <br/> _(전체 시스템 아키텍처 도식화 문서, 핵심 기능 프로세스 다이어그램 문서, ...)_

#### 사용 기술

Kotlin, Armeria(gRPC), Spring Webflux, Hibernate, R2DBC, PostgreSQL, RabbitMQ, AWS, Kubernetes, Helm, ArgoCD, Docker, Microservice Architecture, Hexagonal Architecture, PostGIS, GDAL

<br/>

---

## 경험

### [Devooks (전자책 판매 서비스)](https://github.com/LeeSM0518/devooks)

_2024.06 ~ 개발중_

#### 기여

- 프로젝트 리더 (프론트엔드 개발자 1명과 협업)
- 회원/인증/전자책/알림/PDF 등 모든 기능 설계 및 구현 (약 50개의 API)
- 모든 기능들에 대한 153개의 테스트 구현
- 테스트 수행 후 AWS로 자동 배포하는 CI/CD 구축
- AWS EC2를 활용하여 개발 서버 운영중
- 네이티브 쿼리를 JOOQ로 개선

#### 사용 기술

Kotlin, Spring Webflux, R2DBC, JOOQ, Flyway, PostgreSQL, AWS

<br/>

### Coinliveview (가상화폐 커뮤니티 어플)

_2023.12 ~ 2025.03_

#### 기여

- 프로젝트 리더 (어플 개발자 1명, 기획자 1명과 협업)
- 전화인증/회원/게시글/팔로우 등 어플에서 사용되는 모든 기능 설계 및 구현
- 모든 기능들에 대한 143개의 테스트 구현
- AWS EC2를 활용하여 서비스 운영 경험
- OneSignal을 활용한 푸시 알림 프로세스 설계 및 구현

#### 사용 기술

Kotlin, Spring Webflux, R2DBC, PostgreSQL, AWS

<br/>

### [Model Factory (웹 기반 딥러닝 서비스)](https://github.com/2020-capstone-project/model-factory)

_2020.01 ~ 2020.09_

#### 기여

- 프로젝트 리더
- 사용자의 딥러닝 요청 처리의 전체 프로세스 설계 및 구현
- 딥러닝 현황 모니터링 프로세스 설계 및 구현
- 사용자에게 딥러닝을 편리하게 제공하기 위한 모든 UI/UX 설계 및 구현

#### 사용 기술

Java, Spring, MyBatis, PostgreSQL, Python, Flask, Keras, JavaScript, Vue

<br/>

---

<div id="tail-wrapper"></div>

<script type="text/javascript">
  (function () {
    const origin = 'https://giscus.app';
    const lightTheme = 'light';
    const darkTheme = 'dark_dimmed';

    let initTheme = lightTheme;
    const html = document.documentElement;

    if (
      (html.hasAttribute('data-mode') &&
        html.getAttribute('data-mode') === 'dark') ||
      (!html.hasAttribute('data-mode') &&
        window.matchMedia('(prefers-color-scheme: dark)').matches)
    ) {
      initTheme = darkTheme;
    }

    let lang = '{{ site.comments.giscus.lang | default: lang }}';
    {%- comment -%} https://github.com/giscus/giscus/tree/main/locales {%- endcomment -%}
    if (lang.length > 2 && !lang.startsWith('zh')) {
      lang = lang.slice(0, 2);
    }

    let giscusAttributes = {
      src: 'https://giscus.app/client.js',
      'data-repo': '{{ site.comments.giscus.repo}}',
      'data-repo-id': '{{ site.comments.giscus.repo_id }}',
      'data-category': '{{ site.comments.giscus.category }}',
      'data-category-id': '{{ site.comments.giscus.category_id }}',
      'data-mapping': '{{ site.comments.giscus.mapping | default: 'pathname' }}',
      'data-strict' : '{{ site.comments.giscus.strict | default: '0' }}',
      'data-reactions-enabled': '{{ site.comments.giscus.reactions_enabled | default: '1' }}',
      'data-emit-metadata': '0',
      'data-theme': initTheme,
      'data-input-position': '{{ site.comments.giscus.input_position | default: 'bottom' }}',
      'data-lang': lang,
      'data-loading': 'lazy',
      crossorigin: 'anonymous',
      async: ''
    };

    let giscusScript = document.createElement('script');
    Object.entries(giscusAttributes).forEach(([key, value]) =>
      giscusScript.setAttribute(key, value)
    );
    document.getElementById('tail-wrapper').appendChild(giscusScript);

    addEventListener('message', (event) => {
      if (
        event.source === window &&
        event.data &&
        event.data.direction === ModeToggle.ID
      ) {
        {%- comment -%} global theme mode changed {%- endcomment -%}
        const mode = event.data.message;
        const theme = mode === ModeToggle.DARK_MODE ? darkTheme : lightTheme;

        const message = {
          setConfig: {
            theme: theme
          }
        };

        const giscus = document.getElementsByClassName('giscus-frame')[0].contentWindow;
        giscus.postMessage({ giscus: message }, origin);
      }
    });
  })();
</script>
