---
title: 첫 셸 스크립트 만들기 (feat. 자동 로깅 스크립트)
date: 2022-03-26
categories: [TIL, Opensource]
tags: [shell, bash, log] # TAG names should always be lowercase
image:
  src: https://noticon-static.tammolo.com/dgggcrkxq/image/upload/v1640252186/noticon/imqc7seefsupvwbjdp94.png
  width: 100 # in pixels
  height: 100 # in pixels
  alt: Bash
mermaid: true
published: true
excerpt_separator: <!--end-of-description-->
---

## 💁 설명

요새 [15분](https://github.com/datalater/15) 동안 다양한 주제의 문서를 읽고 요약하는 시간을 갖고 있습니다. 주제별로 디렉토리 구조를 만들어서 정리하고 있어요. 그런데 분류할 때는 편하지만, 그동안 무엇을 읽고 어떤 파일을 작성했는지 한눈에 파악하기는 어려웠습니다. 문서를 추가하거나 수정할 때마다 자동으로 로깅을 해주면 좋겠다는 생각이 들었습니다.

그래서 처음으로 셸 스크립트를 작성해서 자동 로깅을 구현했습니다. 문서를 수정할 때마다 로깅 파일을 남기는 셸 스크립트를 작성하고, pre-commit 훅으로 셸 스크립트를 자동 실행하는 과정을 정리합니다.

<!--end-of-description-->

## 요구사항 정리

```diff
 git 명령어로 현재 스테이징된 파일 목록을 가져온다
 스테이징된 파일에서 첫번째 줄을 읽어서 원문 링크와 요약 파일 링크를 가져온다.
 가져온 데이터를 사용해서 마크다운 테이블의 로우(row)로 만든다.
 로우가 1개 이상이면 LOG.md 파일에 추가(append)한다.
```

## 셸 스크립트 기본 문법 파악하기

> [참조 - Shell Scripting Tutorial](https://www.shellscript.sh/first.html)

### 변수

```bash
#!/bin/sh

# 사용자 입력을 받아서 변수에 저장하고 변수 값으로 파일 만들기
echo "What is your name?"
read USER_NAME
echo "Hello $USER_NAME"
echo "I will create you a file called ${USER_NAME}_file"
touch "${USER_NAME}_file"
```

### 반복문

```bash
#!/bin/sh

# bye라고 입력할 때까지 반복
INPUT_STRING=hello
while [ "$INPUT_STRING" != "bye" ]
do
  echo "Please type something in (bye to quit)"
  read INPUT_STRING
  echo "You typed: $INPUT_STRING"
done

# 명시한 내용으로 반복
for i in hello 1 * 2 goodbye
do
  echo "Looping ... i is set to $i"
done
# Looping ... i is set to hello
# Looping ... i is set to 1
# Looping ... i is set to 101.md
# Looping ... i is set to first.sh
# Looping ... i is set to 2
# Looping ... i is set to goodbye
#
# *는 현재 스크립트를 실행하는 디렉토리 내부에 있는 모든 파일을 가져온다.
```

## 완성된 코드

### script.sh

<!-- prettier-ignore-start -->
```bash
#!/bin/sh

# Added files in git
files=$(git diff --cached --name-only --diff-filter=ACM)

echo "Added files: $files"

# Create LOG.md if it doesn't exist
if ! [[ -f "LOG.md" ]]; then
cat > LOG.md <<- EOF
# 📜 LOG

| Last modified | Article | Summary |
| --- | --- | --- |
EOF
fi

# Make table rows as looping through files
rows=""
newline=$'\n'
for filename in $files
do
  # Skip if file is not markdown
  if ! [[ $filename == *.md ]]; then
    continue
  fi

  if [[ $filename == LOG.md || $filename == README.md ]]; then
    continue
  fi

  headline=$(git blame -L 1,1 $filename)

  if [[ -ne $headline ]]; then
    last_modified=$(git blame $filename | grep -Eo '\b[[:digit:]]{4}-[[:digit:]]{2}-[[:digit:]]{2}\b' | sort -n | tail -1)
    article=$(echo $headline | awk '{split($0, array, "# "); print array[2]}')

    row="| $last_modified | $article | [$filename](./$filename) |"
    rows+=${row}${newline}
  fi
done

# Remove last new line to append new rows next time script is run
rows=${rows%${newline}}
echo "Rows: $rows"

# Append new rows to LOG.md
if [[ -ne $rows ]]; then
cat >> LOG.md <<- EOF
$rows
EOF

git add LOG.md
fi

# Remove file if it doesn't contain any content
lc=$(wc -l LOG.md | awk '{print $1}')
defaultLines=4
if ! [[ $lc -gt $defaultLines ]]; then
  rm LOG.md
fi
```
{: file="script.sh" }
<!-- prettier-ignore-end -->

스크립트에 실행 권한을 추가합니다.

```console
$ chmod 755 script.sh
```

### .git/hooks/pre-commit

git으로 commit을 할 때마다 commit 전에 위 스크립트를 실행하도록 pre-commit 훅을 설정합니다.

먼저 `.git/hooks/pre-commit.sample` 파일명을 변경합니다. 파일명 `pre-commit.sample`에서 `.sample`을 삭제하면 `pre-commit` 훅이 자동으로 실행되기 때문입니다.

```console
$ mv .git/hooks/pre-commit.sample .git/hooks/pre-commit
```

이제 `.git/hooks/pre-commit` 파일을 수정합니다.

<!-- prettier-ignore-start -->
```bash
#!/bin/sh

echo "Running pre-commit hook..."
source ./script.sh
```
{: file=".git/hooks/pre-commit" }
<!-- prettier-ignore-end -->

`pre-commit` 파일은 `.git/hooks/` 디렉토리 내부에 있지만 실행할 때는 프로젝트 루트 경로를 참조하므로 `../../script.sh` 같은 형태가 아닌 `./script.sh`로 작성하면 됩니다.

설정이 완료되었습니다.

전체 코드는 [datalater/15](https://github.com/datalater/15)를 참조해주세요.

## 📚 함께 보기

- [우아한형제들 - 훅으로 Git에 훅 들어가기](https://techblog.woowahan.com/2530/)
- [Shell scripting tutorial](https://www.shellscript.sh/test.html)
- [stackoverflow - creating files with some content with shell script](https://stackoverflow.com/questions/4879025/creating-files-with-some-content-with-shell-script): here document 사용 예시
- [huammmm1 - here document와 here string에 대해서](https://huammmm1.tistory.com/517): here document 사용 예시
- [stackoverflow - get the latest date from a text file](https://stackoverflow.com/questions/57971676/get-the-latest-date-from-a-text-file)
- [stackoverflow - how to call one shell script from another?](https://stackoverflow.com/questions/8352851/shell-how-to-call-one-shell-script-from-another-shell-script)
- [stackoverflow - Git pre-commit hook : changed/added files](https://stackoverflow.com/questions/2412450/git-pre-commit-hook-changed-added-files): 스테이지에 추가된 파일 목록 보기
- [github - datalater/15](https://github.com/datalater/15)
