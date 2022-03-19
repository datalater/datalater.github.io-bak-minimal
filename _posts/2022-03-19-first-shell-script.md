---
title: 첫 셸 스크립트 만들기
date: 2022-03-19
categories: [TIL]
tags: [shell] # TAG names should always be lowercase
image:
  src: https://noticon-static.tammolo.com/dgggcrkxq/image/upload/v1640252186/noticon/imqc7seefsupvwbjdp94.png
  width: 100 # in pixels
  height: 100 # in pixels
  alt: Bash
mermaid: true
published: false
excerpt_separator: <!--end-of-description-->
---

## 💁 설명

요새 15분 동안 다양한 주제의 문서를 읽고 요약하는 시간을 갖고 있습니다. 그런데 주제별로 디렉토리 구조를 만들어서 정리하다 보니 그동안 무엇을 읽고 어떤 파일을 작성했는지 한눈에 파악하기 어려웠습니다. 문서를 추가하거나 수정할 때마다 자동으로 로깅을 해주면 좋겠다고 생각했습니다.

그래서 처음으로 셸 스크립트를 작성해서 자동 로깅을 구현했습니다. 문서를 수정할 때마다 로깅 파일을 남기는 셸 스크립트를 작성하고, pre-commit 훅으로 셸 스크립트를 자동 실행하는 과정을 정리합니다.

<!--end-of-description-->

## 완성된 코드

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

# Remove last new line for appending new rows next time script is run
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

## 📚 함께 보기

- [우아한형제들 - 훅으로 Git에 훅 들어가기](https://techblog.woowahan.com/2530/)
- [Shell scripting tutorial](https://www.shellscript.sh/test.html)
- [stackoverflow - creating files with some content with shell script](https://stackoverflow.com/questions/4879025/creating-files-with-some-content-with-shell-script): here document 사용 예시
- [huammmm1 - here document와 here string에 대해서](https://huammmm1.tistory.com/517): here document 사용 예시
- [stackoverflow - get the latest date from a text file](https://stackoverflow.com/questions/57971676/get-the-latest-date-from-a-text-file)
- [stackoverflow - how to call one shell script from another?](https://stackoverflow.com/questions/8352851/shell-how-to-call-one-shell-script-from-another-shell-script)
- [stackoverflow - Git pre-commit hook : changed/added files](https://stackoverflow.com/questions/2412450/git-pre-commit-hook-changed-added-files): 스테이지에 추가된 파일 목록 보기
