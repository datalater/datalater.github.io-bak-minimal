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
---

## 💁 설명

블로그 글이 추가되면서 틈틈이 지난 글을 되돌아보고 퇴고하는 일이 잦아졌습니다. 그런데 직접 글을 특정해서 찾아가기보다는 랜덤으로 제시해주면 여러 가지 글을 골고루 볼 수 있을 것 같은 생각이 들었습니다.

지킬 블로그에 랜덤 포스트 보기 기능을 추가하면서 어떻게 개발했는지 정리합니다.

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

## 레이아웃 추가하기

`random.md` 파일에 적용할 레이아웃 `random.html`을 추가합니다.

{% raw %}

```html
---
layout: page
# The random post layout
---

<div>...loading</div>
```

{% endraw %}

현재 블로그에서 사용하고 있는 [Chirpy Starter](https://github.com/cotes2020/chirpy-starter)는 `/arhicves` URL로 이동하면 모든 포스트 목록을 보여줍니다. 여기서 모든 포스트 목록의 URL을 파싱한 후 랜덤으로 고른 포스트 URL로 이동합니다.

{% raw %}

```html
---
layout: page
# The random post layout
---

<div>...loading</div>

<script type="text/javascript">
  function main() {
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
  }

  // 현재 페이지가 로드되면 main 함수를 실행합니다.
  window.onload = main;
</script>
```

{% endraw %}

## 📚 함께 읽기

- [stackoverflow - returning HTML with fetch()](https://stackoverflow.com/questions/36631762/returning-html-with-fetch)
