---
title: "[Hugo] GitHub Actions로 Hugo 블로그 자동 빌드 & 배포하기"
date: 2026-02-15
draft: false
category: [development]
subcategories: [github-blog]
tags: [Hugo, Github Blog, Github Actions, CI/CD]
---

AWS Lambda 관련 글을 작성한 지 어언 3년. 오랜만에 돌아와 보니 내 블로그가 Hugo를 사용하고 로컬 VMware에서 빌드하여 커밋했다는 것은 기억나는데... 그래서 레포지토리도 2개라는 것은 기억나는데... 상세 내용이 아예 생각나지 않았다. 그래서 이참에 아예 재개편하여 GitHub Actions를 활용해 소스 푸시만으로 빌드와 배포가 자동으로 이루어지도록 수정했다. 그 방법을 정리해 보려 한다.

<!--more-->

Hugo 블로그를 운영할 때, 로컬에서 `hugo` 명령어로 빌드한 뒤 `public` 폴더를 푸시하는 방식을 사용하는 경우가 많다. 기존 방식은 대략 이런 흐름이다.

1. 마크다운으로 글 작성
2. `hugo` 명령어로 빌드
3. 생성된 `public` 폴더를 커밋 & 푸시 (내 경우에는 이 `public` 폴더를 git submodule로 별도의 배포용 레포지토리와 연결하여 사용하고 있었다)

이 방식은 몇 가지 불편한 점이 있다.

- 로컬에 Hugo가 설치되어 있어야 한다
- 빌드된 `public` 폴더가 저장소에 포함되어 저장소 용량이 커진다
- 빌드를 깜빡하고 소스만 푸시하면 배포가 안 된다
- 여러 기기에서 작업할 때 환경 통일이 번거롭다

GitHub Actions를 사용하면 이 문제들이 모두 해결된다. 소스 코드만 관리하면 되고, 빌드는 GitHub이 알아서 해준다.

## 사전 준비

### 1. GitHub Pages 소스를 GitHub Actions로 변경

GitHub 저장소에서 **Settings > Pages > Build and deployment** 항목으로 이동한다. **Source**를 `Deploy from a branch`에서 **GitHub Actions**로 변경한다.

이 설정을 변경하지 않으면 워크플로우를 아무리 잘 작성해도 배포가 되지 않으니 반드시 확인해야 한다.

### 2. public 폴더 정리

기존에 `public` 폴더를 커밋하고 있었다면, `.gitignore`에 추가하고 저장소에서 제거한다.

```plain
# .gitignore
public/
resources/
```

이제부터 `public` 폴더는 GitHub Actions가 빌드 시 자동으로 생성하므로 저장소에 포함할 필요가 없다.

### 3. 워크플로우 파일 작성

프로젝트 루트에 `.github/workflows/hugo.yml` 파일을 생성한다.

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.140.2
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb

      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5

      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
          TZ: Asia/Seoul
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

각 부분별 상세 설명은 아래와 같다.

먼저 워크플로우가 트리거되는 조건을 정의한 부분이다.

```yaml
on:
  push:
    branches:
      - main # main 브랜치에 푸시될 때 자동 실행
  workflow_dispatch: # GitHub 웹 Actions 탭에서 수동 실행 가능
```

다음은 워크플로우의 권한을 설정한 부분이다.

```yaml
permissions:
  contents: read # 저장소 소스 코드 읽기
  pages: write # GitHub Pages 배포를 위한 쓰기 권한
  id-token: write # GitHub Pages 배포 인증에 필요
```

동시 실행을 제어하는 부분이다. 여러 커밋을 연속으로 푸시해도 배포가 꼬이지 않도록 제한한다.

```yaml
concurrency:
  group: "pages"
  cancel-in-progress: false # 진행 중인 배포를 취소하지 않고 순서대로 처리
```

여기서부터 `build` 잡이다. Hugo 버전을 명시적으로 지정하는데, 로컬 환경과 동일한 버전을 사용해야 빌드 결과가 일관된다. Hugo는 버전에 따라 동작이 달라질 수 있으므로 고정하는 것을 권장한다. SCSS를 사용하는 테마라면 반드시 `hugo_extended` 버전을 설치해야 한다. 일반 버전에는 SCSS 컴파일 기능이 포함되어 있지 않아 빌드가 실패한다.

```yaml
env:
  HUGO_VERSION: 0.140.2 # 사용할 Hugo 버전 고정
```

소스 코드를 체크아웃하는 단계다.

```yaml
- name: Checkout
  uses: actions/checkout@v4
  with:
    # submodules: recursive # 테마를 git submodule로 관리하는 경우에만 필요. 테마 소스를 직접 포함하고 있다면 이 옵션은 불필요하다.
    fetch-depth: 0 # 전체 git 히스토리. Hugo의 .GitInfo, .Lastmod 기능에 필요
```

실제 Hugo 빌드를 수행하는 단계다.

```yaml
- name: Build with Hugo
  env:
    HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
    HUGO_ENVIRONMENT: production
    TZ: Asia/Seoul # 글의 날짜를 한국 시간 기준으로 처리
  run: |
    hugo \
      --gc \ # 사용하지 않는 캐시 파일 정리
      --minify \ # HTML, CSS, JS 압축으로 로딩 속도 개선
      --baseURL "${{ steps.pages.outputs.base_url }}/" # GitHub Pages URL 자동 주입
```

`--baseURL`은 `configure-pages` 액션에서 제공하는 값을 사용하므로 하드코딩할 필요가 없다.

마지막으로 `deploy` 잡이다. `needs: build`로 빌드가 완료된 후에만 배포가 실행되며, `deploy-pages` 액션이 앞서 업로드된 아티팩트를 GitHub Pages로 배포한다.

```yaml
deploy:
  environment:
    name: github-pages
    url: ${{ steps.deployment.outputs.page_url }}
  runs-on: ubuntu-latest
  needs: build # build 잡 완료 후 실행
  steps:
    - name: Deploy to GitHub Pages
      id: deployment
      uses: actions/deploy-pages@v4
```

## 배포 확인

설정이 완료되면 아래의 흐름으로 동작한다.

1. 마크다운 작성 후 커밋 & 푸시
2. GitHub Actions가 자동으로 빌드 시작
3. 빌드 성공 시 GitHub Pages에 자동 배포

배포 상태는 저장소의 **Actions** 탭에서 확인할 수 있다. 빌드에 실패하면 로그를 통해 원인을 파악할 수 있으며, `workflow_dispatch` 설정이 있으므로 수동으로 재실행도 가능하다.

## 주의할 점

- **Hugo 버전 관리**: 로컬에서 `hugo version`으로 확인한 버전과 워크플로우의 `HUGO_VERSION`을 일치시킨다. 버전이 다르면 로컬에서는 정상인데 배포 시 빌드가 실패할 수 있다.
- **Extended 버전**: SCSS/SASS를 사용하는 테마라면 반드시 `hugo_extended` 버전을 사용해야 한다. 워크플로우에서 다운로드하는 `.deb` 파일명에 `extended`가 포함되어 있는지 확인한다.
- **submodule**: 테마를 git submodule로 관리하지 않고 직접 소스를 포함하고 있다면 `submodules: recursive` 옵션은 없어도 된다.
- **baseURL**: `hugo.toml`(또는 `config.toml`)의 `baseURL`과 실제 GitHub Pages URL이 일치하는지 확인한다. 워크플로우에서 `--baseURL` 플래그로 덮어쓰므로 큰 문제는 없지만, 로컬 테스트 시 혼동될 수 있다.
