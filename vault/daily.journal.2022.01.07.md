---
id: yWkEmaFTLw9g9C3YeQrKf
title: '2022-01-07'
desc: ''
updated: 1641529627327
created: 1641528927434
traitIds:
  - journalNote
---

# Check Lastest Commit

### 代码片段

```sh
echo "${_group}Checking for latest commit ... "

# Checks if we are on latest commit from github if it is running from master branch
if [[ -d "../.git" && "${SKIP_COMMIT_CHECK:-0}" != 1 ]]; then
  if [[ $(git branch | sed -n '/\* /s///p') == "master" ]]; then
    if [[ $(git rev-parse HEAD) != $(git ls-remote $(git rev-parse --abbrev-ref @{u} | sed 's/\// /g') | cut -f1) ]]; then
      echo "Seems like you are not using the latest commit from the self-hosted repository. Please pull the latest changes and try again, or suppress this check with --skip-commit-check.";
      exit 1
    fi
  fi
else
  echo "skipped"
fi

echo "${_endgroup}"

```

### 解释

- `git rev-parse HEAD` 获取本地当前节点 hash 版本号
- `git ls-remote [origin_name] [branch_name]` 获取远程仓库 hash 版本号


