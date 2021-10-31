+++
title = "使用 GitHub Action 來發布 Hugo 網站"
description = ""
date = 2021-10-31T13:17:03+08:00
tags = ["github action", "hugo"]
draft = false
+++

最近重拾翻譯，希望自己有限的時間可以著重在「翻譯」本身，需要經常更新翻譯進度。

之前都是很手工藝要去 Blogger 貼，再自己用 InDesign 製作 epub/PDF，相當原始，花在使用工具的時間比用在翻譯時間多很多。

研究了一番使用 Hugo + GitHub Page 為解決方案，改天再來寫一篇。

這裡先紀錄一下，直接裝 GitHub Action 在 Push master 的時候能夠 Build public files 到 `/docs` 去，才不會有更新段落卻忘記 Build Page 更新網站。

主要 `deploy.yml` 修改來自 [lisez/hugo-auto-deploy.yml](https://gist.github.com/lisez/41cebe4eb9190a5c5e879fee5933beb1)。

我順便連同本 Repo 的更新機制也換掉了。

[原始碼在此](https://github.com/raviwu/raviwu.github.io/blob/master/.github/workflows/deploy.yml)

前置工作：

1. 產一個 [GitHub Personal Access Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
2. 設定 [Repo Secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

```yaml
name: Hugo Publish

on:
  push:
    branches:
    - master

jobs:
  hugo-publish:
     name: publish content to public site
     runs-on: ubuntu-latest
     steps:
       - name: checkout private repo
         uses: actions/checkout@v2
         with:
           token: ${{ secrets.PUSH_REPO_TOKEN }}

       - name: checkout submodules
         shell: bash
         run: |
          auth_header="$(git config --local --get http.https://github.com/.extraheader)"
          git submodule sync --recursive
          git -c "http.extraheader=$auth_header" -c protocol.version=2 submodule update --init --force --recursive --depth=1
       - name: setup hugo
         uses: peaceiris/actions-hugo@v2
         with:
           hugo-version: latest
           extended: true

       - name: build content to public site
         run: hugo --minify --gc -t tale

       - name: deploy and publish updates
         run: |
           git config --local user.email "action@github.com"
           git config --local user.name "GitHub Action"
           git add . -A
           git commit -m "[chore] Auto publish"
           git push origin
```

之後就可以專心寫作，推上 GitHub 後 Action 會接續發布的事宜。要記得往後得 `git pull` 才能把 github action 推上 master 的檔案同步到本機 repo。