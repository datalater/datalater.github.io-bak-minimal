# Cheo's Devlog Guide

## 글 작성법

### front matter 작성법

```markdown
---
title: emotion theme 타입 지정하는 방법
date: 2021-12-04
categories: [TIL]
tags: [typescript] # TAG names should always be lowercase
image:
  src: https://noticon-static.tammolo.com/dgggcrkxq/image/upload/v1566913457/noticon/eh4d0dnic4n1neth3fui.png
  width: 100 # in pixels
  height: 100 # in pixels
  alt: TypeScript
---
```

이미지는 [noticon](https://noticon.tammolo.com/)에서 검색하여 사용한다.

### 목차 자동 생성을 위한 헤더 작성법

프론트 매터에 있는 title이 h1이 되므로 본문에서는 h2 이하로 작성하면 된다.

### 이미지 삽입 방법

- 이미지를 [Github Issue](https://github.com/datalater/datalater.github.io/issues/1)에 업로드하고, 생성된 이미지 주소를 입력한다.

```markdown
![error](https://user-images.githubusercontent.com/8105528/144628239-faf1e84a-26ec-49d0-8dab-23f4c81af527.png){: .shadow }
_theme에서 타입 오류가 발생한다_
```

### Prettier ignore

테이블 정렬하는 자동 포맷 비활성화:

```markdown
**ESLint 패키지**

{{<!-- prettier-ignore -->}}
| 용어 | 뜻 | 예시 |
| :---: | :---: | :---: |
| eslint-plugin-\* | ESLint 커스텀 규칙을 정의한 패키지 | [`eslint-plugin-react`](https://github.com/yannickcr/eslint-plugin-react) |
| eslint-config-\* | 커스텀 규칙을 포함하여 ESLint 설정 자체를 정의한 패키지 {::nomarkdown}</br>{:/} (eslint-config-\*를 포함하기도 한다) | [`eslint-config-airbnb`](https://www.npmjs.com/package/eslint-config-airbnb) |
```

파일명과 코드블록 사이에 개행 넣는 자동 포맷 비활성화:

````markdown
**린트 설정 파일 작성:**

<!-- prettier-ignore-start -->
```js
module.exports = {
  root: true,
  parser: "@typescript-eslint/parser",
  plugins: ["@typescript-eslint"],
  extends: ["eslint:recommended", "plugin:@typescript-eslint/recommended"],
};
```
{: file=".eslintrc.js" }
<!-- prettier-ignore-end -->
````

## 서버 실행 방법

```bash
# Gemfile 의존성 설치
bundle install

# 로컬 서버 실행
bundle exec jekyll serve
```

## 이 블로그는 어떻게 생성되었나

[Chirpy Starter](https://github.com/cotes2020/chirpy-starter)를 사용하여 생성되었다.

## 블로그를 커스텀으로 변경하는 방법

> [참조 - Chirpy Starter](https://github.com/cotes2020/chirpy-starter)

```bash
# 스타터로 만든 실제 파일 경로를 출력한다
$ bundle info --path jekyll-theme-chirpy

# 위에서 출력한 경로로 이동한다
$ cd /Users/cheo/.rbenv/versions/3.0.3/lib/ruby/gems/3.0.0/gems/jekyll-theme-chirpy-4.3.4

# 예를 들어, uttrances 댓글을 넣으려면 _layouts/post.html을 수정해야 하므로
# jekyll-theme-chirpy-4.3.4에 있는 _layouts/ 폴더를 블로그 프로젝트 폴더로 복사한다
$ rsync -av _layouts/ ~/datalater/datalater.github.io/_layouts

# _layouts/post.html에서 disqus 대신에 utterances에서 제공하는 스크립트 태그를 넣는다.
```
