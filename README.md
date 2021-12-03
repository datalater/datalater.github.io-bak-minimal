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

## 서버 실행 방법

```bash
# Gemfile 의존성 설치
bundle install

# 로컬 서버 실행
bundle exec jekyll serve
```

## 환경 설정

````markdown
```shell
# content
```
{: file="path/to/file" }
````

위와 같이 코드 블록에 파일명을 명시하려면 `{: file="path/to/file" }`가 코드 블록 닫는 줄(` ``` `)의 바로 다음 줄에 있어야 한다. 그런데 `prettier`를 사용하면 자동으로 개행을 시키므로 이를 비활성화해야 한다. 따라서 `.vscode/settings.json` 파일을 생성하여 마크다운 파일은 기본 포매터를 `null`로 설정한다.

## 이 블로그는 어떻게 생성되었나

[Chirpy Starter](https://github.com/cotes2020/chirpy-starter)를 사용하여 생성되었다.

