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

요새 [15분](https://github.com/datalater/15) 동안 다양한 주제의 문서를 읽고 요약하는 시간을 갖고 있습니다. 주제별로 디렉토리 구조를 만들어서 정리하고 있어요. 그런데 개별 파일을 분류할 때는 편하지만, 그동안 무엇을 읽고 어떤 파일을 작성했는지 전체를 한눈에 파악하기는 어려웠습니다. 문서를 추가하거나 수정할 때마다 자동으로 로깅을 해주면 좋겠다는 생각이 들었습니다.

그래서 처음으로 셸 스크립트를 작성해서 자동 로깅을 구현했습니다. 문서를 수정할 때마다 로깅 파일을 남기는 셸 스크립트를 작성하고, pre-commit 훅으로 셸 스크립트를 자동 실행하는 과정을 정리합니다.

<!--end-of-description-->

## 요구사항 정리

```markdown
자동 로깅 스크립트

- [ ] git 명령어로 현재 스테이징된 파일 목록을 가져온다.
- [ ] 스테이징된 파일에서 첫번째 줄을 읽어서 원문 링크와 요약 파일 링크를 가져온다.
- [ ] 가져온 데이터를 사용해서 마크다운 테이블의 로우(row)로 만든다.
- [ ] 로우가 1개 이상이면 LOG.md 파일에 추가(append)한다.

pre-commit 훅

- [ ] 자동 로깅 스크립트를 실행하는 pre-commit 훅을 만든다.
- [ ] pre-commit 훅이 프로젝트 저장소에 저장되도록 설정한다.
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

# new line이 들어간 문자열 변수로 반복
files="LOG.md\nfrontend/css-layout-algorithm.md\nlog.sh"
echo "$files" | while IFS= read -r filename; do
  if ! [[ -z "$filename" ]]; then
    echo "- $filename"
  fi
done
```

### 조건문

```bash
#!/bin/sh

# 조건문 구조
if  [ something ]; then
 echo "Something"
 elif [ something_else ]; then
   echo "Something else"
 else
   echo "None of the above"
fi

# 비교
if [ "$X" -lt "0" ]; then
  echo "X is less than zero"
fi

if [ "$X" -gt "0" ]; then
  echo "X is more than zero"
fi

if [ -z "$var" ]; then
  echo "\$var is empty"
else
  echo "\$var is NOT empty"
fi

# &&과 ||으로 조건문 짧게 쓰기
[ "$X" -le "0" ] && \
      echo "X is less than or equal to  zero"
[ "$X" -ge "0" ] && \
      echo "X is more than or equal to zero"
[ "$X" = "0" ] && \
      echo "X is the string or number \"0\""
[ "$X" = "hello" ] && \
      echo "X matches the string \"hello\""
[ "$X" != "hello" ] && \
      echo "X is not the string \"hello\""
[ -n "$X" ] && \
      echo "X is of nonzero length"
[ -f "$X" ] && \
      echo "X is the path of a real file" || \
      echo "No such file: $X"
[ -x "$X" ] && \
      echo "X is the path of an executable file"
[ "$X" -nt "/etc/passwd" ] && \
      echo "X is a file which is newer than /etc/passwd"
```

## 완성된 코드

### log.sh

<!-- prettier-ignore-start -->
```bash
#!/bin/sh

# Only Added(A) files in git (Not modified files)
files=$(git diff --cached --name-only --diff-filter=A)

if ! [[ -z "$files" ]]; then
  echo "Added files:"
  echo "$files" | while IFS= read -r filename; do 
    if ! [[ -z "$filename" ]]; then
      echo "- $filename"
    fi
  done
  echo ""
fi

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
echo "$files" | while IFS= read -r filename; do 
  # Skip if file is not markdown
  if ! [[ $filename == *.md ]]; then
    continue
  fi

  if [[ $filename == LOG.md || $filename == README.md ]]; then
    continue
  fi

  headline=$(git blame -L 1,1 $filename)
  if ! [[ -z $headline ]]; then
    last_modified=$(git blame $filename | grep -Eo '\b[[:digit:]]{4}-[[:digit:]]{2}-[[:digit:]]{2}\b' | sort -n | tail -1)
    article=$(echo $headline | awk '{split($0, array, "# "); print array[2]}')

    # Skip if file is already in LOG.md (we will log only the first time the file is added)
    alreadyExists=$(grep -Fc "$article" LOG.md)

    if [[ $alreadyExists -gt 0 ]]; then
      echo "Skipping already existing $article..."
      continue;
    fi

    row="| $last_modified | $article | [$filename](./$filename) |"
    rows+=${row}${newline}
  fi
done

# Remove last new line for appending new rows next time script is run
rows=${rows%${newline}}

if ! [[ -z $rows ]]; then
  echo "Rows: $rows"
fi

# Append new rows to LOG.md
if ! [[ -z $rows ]]; then
cat >> LOG.md <<- EOF
$rows
EOF

git add LOG.md
fi

# Remove file if LOG file is an initialized default file
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
$ chmod +x script.sh
```

### pre-commit hook

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
source ./log.sh
```
{: file=".git/hooks/pre-commit" }
<!-- prettier-ignore-end -->

> `pre-commit` 파일은 `.git/hooks/` 디렉토리 내부에 있지만 실행할 때는 프로젝트 루트 경로를 참조하므로 `../../setup.sh` 같은 형태가 아닌 `./setup.sh`로 작성하면 됩니다.

### git config를 프로젝트에 포함시키기

`.git` 폴더에 있는 `.git/hooks/pre-commit` 파일은 프로젝트 저장소에 버전 관리되지 않습니다. 따라서 현재 컴퓨터에서는 pre-commit 훅이 동작하더라도 다른 컴퓨터에서 이 프로젝트를 클론하면 위에서 설정한 훅이 사라지게 됩니다. 훅 폴더를 새로 만들고 git config의 훅의 경로(core.hooksPath)가 새로 만든 폴더를 바라보도록 설정하겠습니다.

```bash
# 1. 프로젝트 루트 경로에서 .git-hooks 폴더를 생성합니다
md .git-hooks

# 2. .git-hooks 폴더에 pre-commit 훅 파일을 생성합니다.
cat > .git-hooks/pre-commit <<- EOF
#!/bin/sh

echo "Running pre-commit hook..."
source ./log.sh
EOF

# 3. 스크립트에 실행 권한을 추가합니다.
chmod +x .git-hooks/pre-commit

# 4. git config 명령어로 hooks 경로가 .git-hooks 폴더를 바라보도록 설정합니다.
git config core.hooksPath .git-hooks
```

여기서 1~3번의 결과물은 프로젝트 파일로 포함되지만 4번 명령어는 프로젝트를 클론할 때마다 기억하고 반복해야 합니다. 스크립트로 작성하여 자동화하겠습니다.

<!-- prettier-ignore-start -->
```bash
cat > setup.sh <<- EOF
#!/bin/sh

git config core.hooksPath .git-hooks
EOF
```
<!-- prettier-ignore-end -->

이제 명령어를 기억할 필요없이 `source setup.sh`를 실행하면 git config에서 hooks 경로가 `.git-hooks` 폴더를 바라보도록 설정됩니다. 바로 실행해줍시다.

```bash
chmod +x setup.sh
source setup.sh
```

설정이 완료되었습니다. 전체 코드는 [datalater/15](https://github.com/datalater/15)를 참조해주세요.

## 📚 함께 보기

- [우아한형제들 - 훅으로 Git에 훅 들어가기](https://techblog.woowahan.com/2530/)
- [Shell scripting tutorial](https://www.shellscript.sh/test.html)
- [stackoverflow - creating files with some content with shell script](https://stackoverflow.com/questions/4879025/creating-files-with-some-content-with-shell-script): here document 사용 예시
- [huammmm1 - here document와 here string에 대해서](https://huammmm1.tistory.com/517): here document 사용 예시
- [stackoverflow - get the latest date from a text file](https://stackoverflow.com/questions/57971676/get-the-latest-date-from-a-text-file)
- [stackoverflow - how to call one shell script from another?](https://stackoverflow.com/questions/8352851/shell-how-to-call-one-shell-script-from-another-shell-script)
- [stackoverflow - Git pre-commit hook : changed/added files](https://stackoverflow.com/questions/2412450/git-pre-commit-hook-changed-added-files): 스테이지에 추가된 파일 목록 보기
- [stackoverflow - how to test if string exists in a file with bash](https://stackoverflow.com/questions/4749330/how-to-test-if-string-exists-in-file-with-bash): 파일에 문자열 포함되어 있는지 확인하기
- [github - datalater/15](https://github.com/datalater/15)
