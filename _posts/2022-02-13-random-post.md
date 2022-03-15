---
title: 지킬 블로그에 랜덤 포스트 보기 기능 추가하기
date: 2022-02-15
categories: [TIL]
tags: [jekyll] # TAG names should always be lowercase
image:
  src: https://jekyllrb.com/img/logo-2x.png
  width: 100 # in pixels
  height: 100 # in pixels
  alt: Jekyll
mermaid: true
published: true
excerpt_separator: <!--end-of-description-->
---

## 💁 설명

블로그 글이 추가되면서 틈틈이 지난 글을 되돌아보고 퇴고하는 일이 잦아졌습니다. 그런데 직접 글을 특정해서 찾아가기보다는 랜덤으로 제시해주면 여러 가지 글을 골고루 볼 수 있을 것 같은 생각이 들었습니다.

지킬 블로그에 랜덤 포스트 보기 기능을 추가하면서 어떻게 개발했는지 정리합니다.

<!--end-of-description-->

<details>
<summary><strong>TL;DR</strong></summary>

모든 포스트 URL을 보여주는 동일한 도메인의 리소스를 크롤링하여 랜덤 URL로 이동하는 방식으로 기능을 구현했습니다.

</details>

## 사이드바에 랜덤 메뉴 아이템 추가하기

`_tabs` 디렉토리에 `random.md` 파일을 생성합니다.

<!-- prettier-ignore-start -->
````md
---
layout: random
title: Random
icon: fas fa-info
order: 5
---
````
{: file="_tabs/random.md" }
<!-- prettier-ignore-end -->

## 사이드바에서 랜덤 메뉴에 대한 클릭 이벤트 핸들러 등록

> ⚠️ Update
>
> - 2022.3.15

`_includes/sidebar.html`에 다음 코드를 추가합니다:

```html
<!-- ...중략... -->

<!-- random post -->
<script type="text/javascript">
  const random = document.querySelector("[href*='/random']");

  if (random) {
    random.addEventListener("click", (e) => {
      e.preventDefault();

      fetch("/archives")
        .then((response) => response.text())
        .then((htmlText) => {
          const parser = new DOMParser();
          const document = parser.parseFromString(htmlText, "text/html");

          const posts = document.querySelectorAll("#archives [href*=posts]");
          const links = Array.from(posts).map((post) => post.href);

          const randomIndex = Math.floor(Math.random() * links.length);
          const randomLink = links[randomIndex];

          window.location = randomLink;
        });
    });
  }
</script>
```

<details markdown="1">
<summary><strong>sidebar.html 코드 전문</strong></summary>

```html
<!--
  The Side Bar
-->

<div id="sidebar" class="d-flex flex-column align-items-end" lang="{{lang}}">
  <div class="profile-wrapper text-center">
    <div id="avatar">
      <a href="{{ '/' | relative_url }}" alt="avatar" class="mx-auto">
        {% if site.avatar != '' and site.avatar %}
          {% capture avatar_url %}
            {%- if site.avatar contains '://' -%}
              {{ site.avatar }}
            {%- elsif site.img_cdn != '' and site.img_cdn -%}
              {{ site.avatar | prepend: site.img_cdn }}
            {%- else -%}
              {{ site.avatar | relative_url }}
            {%- endif -%}
          {% endcapture %}
          <img src="{{ avatar_url }}" alt="avatar" onerror="this.style.display='none'">
        {% endif %}
      </a>
    </div>

    <div class="site-title mt-3">
      <a href="{{ '/' | relative_url }}">{{ site.title }}</a>
    </div>
    <div class="site-subtitle font-italic">{{ site.tagline }}</div>

  </div><!-- .profile-wrapper -->

  <ul class="w-100">

    <!-- home -->
    <li class="nav-item{% if page.layout == 'home' %}{{ " active" }}{% endif %}">
      <a href="{{ '/' | relative_url }}" class="nav-link">
        <i class="fa-fw fas fa-home ml-xl-3 mr-xl-3 unloaded"></i>
        <span>{{ site.data.locales[lang].tabs.home | upcase }}</span>
      </a>
    </li>
    <!-- the real tabs -->
    {% for tab in site.tabs %}
    <li class="nav-item{% if tab.url == page.url %}{{ " active" }}{% endif %}">
      <a href="{{ tab.url | relative_url }}" class="nav-link">
        <i class="fa-fw {{ tab.icon }} ml-xl-3 mr-xl-3 unloaded"></i>
        {% capture tab_name %}{{ tab.url | split: '/' }}{% endcapture %}

        <span>{{ site.data.locales[lang].tabs.[tab_name] | default: tab.title | upcase }}</span>
      </a>
    </li> <!-- .nav-item -->
    {% endfor %}

  </ul> <!-- ul.nav.flex-column -->

  <div class="sidebar-bottom mt-auto d-flex flex-wrap justify-content-center align-items-center">

    {% for entry in site.data.contact %}
      {% capture url %}
        {%- if entry.type == 'github' -%}
          https://github.com/{{ site.github.username }}
        {%- elsif entry.type == 'twitter' -%}
          https://twitter.com/{{ site.twitter.username }}
        {%- elsif entry.type == 'email' -%}
          {% assign email = site.social.email | split: '@' %}
          javascript:location.href = 'mailto:' + ['{{ email[0] }}','{{ email[1] }}'].join('@')
        {%- elsif entry.type == 'rss' -%}
          {{ "/feed.xml" | relative_url }}
        {%- else -%}
          {{ entry.url }}
        {%- endif -%}
      {% endcapture %}

      {% if url %}
      <a href="{{ url }}" aria-label="{{ entry.type }}"
        {% unless site.theme_mode %}class="order-{{ forloop.index | plus: 2 }}"{% endunless %}
        {% unless entry.noblank %}target="_blank" rel="noopener"{% endunless %}>
        <i class="{{ entry.icon }}"></i>
      </a>
      {% endif %}

    {% endfor %}

    {% unless site.theme_mode %}
      {% if site.data.contact.size > 0 %}
        <span class="icon-border order-2"></span>
      {% endif %}

      <span id="mode-toggle-wrapper" class="order-1">
        {% include mode-toggle.html %}
      </span>
    {% endunless %}

  </div> <!-- .sidebar-bottom -->

</div><!-- #sidebar -->

<!-- random post -->
<script type="text/javascript">
  const random = document.querySelector("[href*='/random']");

  if (random) {
    random.addEventListener('click', (e) => {
      e.preventDefault();

      fetch("/archives")
        .then((response) => response.text())
        .then((htmlText) => {
          const parser = new DOMParser();
          const document = parser.parseFromString(htmlText, "text/html");

          const posts = document.querySelectorAll("#archives [href*=posts]");
          const links = Array.from(posts).map((post) => post.href);

          const randomIndex = Math.floor(Math.random() * links.length);
          const randomLink = links[randomIndex];

          window.location = randomLink;
        })
    })
  }

</script>

```

</details>

---

> ⚠️ DEPREATED
>
> 위 내용이 추가됨에 따라 아래 방식은 더 이상 사용되고 있지 않습니다.

## 레이아웃 추가하기

`random.md` 파일에 레이아웃을 적용하기 위해 `_layouts` 디렉토리에 `/random.html` 파일을 추가합니다.

<!-- prettier-ignore-start -->
{% raw %}
```html
---
layout: page
# The random post layout
---

<div>...loading</div>
```
{: file="_layouts/random.html" }
{% endraw %}
<!-- prettier-ignore-end -->

> 랜덤 포스팅을 불러오기 전에 로딩 메시지 `...loading`을 보여줍니다.

현재 블로그에서 사용하고 있는 [Chirpy Starter](https://github.com/cotes2020/chirpy-starter)는 `/arhicves` URL로 이동하면 모든 포스트 목록을 보여줍니다. `/archives` 리소스를 사용하여 모든 포스트 목록의 URL을 파싱한 후 랜덤으로 고른 포스트 URL로 이동하는 코드를 [즉시 실행 함수(IIFE)](https://developer.mozilla.org/ko/docs/Glossary/IIFE)로 작성합니다.

<!-- prettier-ignore-start -->
{% raw %}
```html
---
layout: page
# The random post layout
---

<div>...loading</div>

<script type="text/javascript"> // here
  (function main() {
    // 모든 포스트 URL 목록이 나오는 /archives 리소스를 파싱합니다.
    fetch("/archives")
      .then((response) => response.text())
      .then((htmlText) => {
        // html text를 DOM 파서를 이용하여 파싱합니다.
        const parser = new DOMParser();
        const document = parser.parseFromString(htmlText, "text/html");

        // href에 /posts/* 를 포함하고 있는 모든 요소를 찾습니다.
        const posts = document.querySelectorAll("#archives [href*=posts]");
        const links = Array.from(posts).map((post) => post.href);

        // 링크 배열에서 랜덤 인덱스를 선택한 후 해당 포스트 URL로 이동합니다.
        const randomIndex = Math.floor(Math.random() * links.length);
        const randomLink = links[randomIndex];

        window.location = randomLink;
      });
  })();
</script>
```
{: file="_layouts/random.html" }
{% endraw %}
<!-- prettier-ignore-end -->

> fetch 메서드를 실행할 때 동일한 오리진의 리소스를 가져오므로 SOP(Same Origin Policy)를 준수합니다. 따라서 CORS(Cross Origin Resource Sharing)를 따로 허용해줄 필요가 없습니다.

## 📚 함께 읽기

- [stackoverflow - returning HTML with fetch()](https://stackoverflow.com/questions/36631762/returning-html-with-fetch)
