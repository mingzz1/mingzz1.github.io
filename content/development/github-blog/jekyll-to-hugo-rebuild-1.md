---
title: "[Hugo] Jekyll to Hugo 블로그 이전기 1"
date: 2021-08-02
draft: false
category: [development]
subcategories: [github-blog]
tags: [Hugo, Github Blog]
---

이 전에도 여러 번 거론했다시피, 내 블로그는 원래 `Jekyll`로 만들었다.
그런데 블로그를 개설한지 어언 3년...
이러저러한 이유 때문에 블로그 프레임워크를 Hugo로 변경하게 되었다.
그래서 변경하게 된 이유와 블로그 구축기를 기록하려 한다.

<!--more-->

### Github + Jekyll 블로그의 시작

내가 블로그를 만든 것은 2018년 초이다.
원래 `Tistory`를 쓰고 있었는데, 

1. 테마가 다양하지 않고
2. 왜인지 `*.github.io`가 간지 나 보여서

`Github`에 블로그를 구축했었다.
그때 당시 이미 `*.github.io` 블로그에 쓸 수 있는 프레임워크에 `Hugo`와 `Jekyll`이 존재했었다.
둘 중 `Jekyll`을 선택 한 이유는 아래와 같았었다.

1. github 블로그로 검색했을 때 Jekyll의 자료가 훨씬 많았다.
2. 1의 이유인지 모르겠으나, 테마도 당연히 Jekyll이 더 많았다.
3. Jekyll은 _post 디렉토리 아래에 md 파일을 만들어서 올라면 됬으나 Hugo는 (그때 당시 생각하기에) 굉장히 복잡한 과정을 거쳐야 해서 어렵다고 느껴졌다.

그런데 `Jekyll`로 구축을 하고 3년 3개월 간 운영을 해보니 여러 단점이 있어 대대적으로 블로그를 개편하면서 `Hugo`로 이전까지 하게 되었다.


### Jekyll의 단점

3년이 넘는 시간동안 사용해 본 `Jekyll`의 단점 중 가장 큰 것은 아무래도 `속도` 문제이다.
오늘 기준으로 내 블로그에는 128개의 글이 있다.
그런데 고작 128 개의 글이 있지만 새로운 글을 올릴 경우 약 3~5분 후에 블로그에 실제로 반영된다.
빌드하는데 시간이 매우 오래 걸린다는 의미이다. 
또한 디자인을 변경하기 위해 `scss` 파일을 변경 할 경우 속도 저하는 더욱 심하다.
때문에 디자인을 변경할 때는 가상머신에 Jekyll을 설치하여 가상머신에서 모든 디자인을 변경한 후 커밋을 해왔다.
그런데 로컬에서 디자인을 변경하더라도 1분 가량 시간이 소요되었으며, 커밋 후에 실제로 반영되기 까지는 약 5분 넘게 걸리기 때문에 사용하기가 매우 불편했다.

반면 `Hugo`의 경우 `빌드 용 Repository`, `빌드 후 실제로 보여지는 블로그 Repository` 이렇게 두 Repository를 관리해야 하는 단점은 있으나, 빌드 시간은 현재 기준 `486ms` 이고 커밋 후 반영까지도 몇 초 정도면 된다.
(거의 즉시 반영이 된다.)
블로그를 처음 만들 때 이미 `속도 차이`에 대한 글을 보긴 했었으나, 내가 이 블로그를 3년동안이나 운영할 줄은 꿈에도 몰랐다... ㅎ
사실 글의 수가 적을 때는 속도 저하를 체감할 수 없을 정도이기 때문에 체감할 수 있는 속도 저하가 나타날 때까지 내가 블로그를 운영할 지 안할 지 모르는 상황에서 진입 장벽이 상대적으로 높아보였던 `Hugo`를 선택하긴 싫었다.

또 다른 단점으로는 블로그를 수정하기 위해 로컬에서 블로그를 세팅했을 때 `bundle install` 하면 에러들이 쏟아졌는데, 이 오류들은 대부분 `Gemfile` 내에 있는 `Gem` 들의 버전이 낮아 Jekyll이나 Gem 중 하나라도 최신 버전으로 깔면 나타나는 오류였다.
최신 버전으로 업데이트 하면 해결할 수 있겠지만 그렇게 되면 `Gemfile` 내의 모든 `Gem` 들을 업데이트 해야하는데 이 때에도 `Gem` 간의 호환성 때문에도 오류가 많이 발생했고, 버전이 바뀌며 사용법도 바뀌어 설정도 손대야 할 부분이 한두개가 아니었다.
때문에 어쩔 수 없이 `Jekyll` 뿐만 아니라 `Gem`도 낮은 버전으로 사용해야 했다.

### Jekyll to Hugo 이전

위의 이유들 때문에 [동생 블로그](https://skye3133.github.io/)를 만들 겸 + 내 블로그를 리뉴얼 할 겸 둘이 협업(?)해서 동생은 블로그의 디자인을 담당하고 나는 Jekyll -> Hugo로 이전을 담당하여 마침내 `mingzzi's blog v.3`이 탄생하게 되었다. ㅎㅎ 
이제부터 블로그 프레임워크를 이전하며 겪은 이슈와 알게 된 내용들을 차근차근 정리하려 한다.

먼저 `Hugo`로 이전하면서 `이것만은 꼭 있어야 한다!` 했던 부분들은 아래와 같다.

* 현재 Jekyll에서의 포스팅의 주소가 Hugo로 이전 한 후에도 동일할 것  
    - 내 경우 Google Adsense, Search Console, Analytics가 붙어있기 때문에 포스팅의 주소가 변경 될 경우 한동안 주소가 변경된 포스팅들을 관리해야 한다. 2년 전에 해봤는데 검색을 통한 유입이 뚝 떨어지기도 하고 매우 귀찮다.. ㅠ
* 태그를 통해 포스팅들을 구분하여 확인할 수 있을 것
    - 현재 블로그에서 태그를 사용하여 포스팅을 구분짓는데, 이렇게 해 두니까 같은 게시물들을 구분하기도 쉽고 메뉴-서브메뉴 구조에서 다른 메뉴로 구분 되더라도 비슷한 성향의 글(ex - Write up)을 모아볼 수 있어서 유용했다. 그래서 이 부분은 유지하고 싶었다.
* 메뉴-서브메뉴 구조를 사용할 수 있을 것
    - 지금 사용하는 블로그에도 나중에 추가한 기능이긴 한데, 메뉴-서브메뉴를 추가하니 시리즈로 있는 글을 모아보기가 훨씬 편했기 때문에 이 부분도 포기할 수 없었다.

가장 중요했던 것이 이정도였던 것 같은데, 어떻게 구현했는지는 차차 적어갈 예정이다. 

#### 1. hugo 환경 세팅

나는 가상머신에서 작업을 했다.
리눅스 환경의 가상머신을 구축하고 아래의 명령어를 입력하면 `Hugo`를 사용할 수 있는 환경이 만들어 진다.
각자 환경에 맞는 설치 방법은 https://gohugo.io/getting-started/installing/를 참고하면 된다.

```bash
$ apt install -y hugo
```

#### 2-1. Jekyll to Hugo 마이그레이션

`Hugo` 설치가 완료되었다면 `Jekyll`에서 `Hugo`로 블로그를 이전할 수 있다.
이 때 간단하게 사용할 수 있는 명령어가 `hugo import jekyll` 이다.

```bash
$ hugo import jekyll [JEKYLL_ROOT_PATH] [HUGO_TARGET_PATH]
```

이 명령어를 사용하면 간단히 `Jekyll`에서 `Hugo`로 블로그 프레임워크를 변경할 수 있다.
디렉토리 구조를 포함하여 어지간한 설정도 함께 변경을 해 주지만, 앞서 말했던 내가 원하는 조건들을 넣기에는 거의 싹 다 뜯어 고쳐야 하는 수준이라서 그냥 새로 사이트를 생성하고 이참에 테마도 새로 생성 해 정말 `나만의 블로그`를 만들기로 했다!

#### 2-2. 새로운 Hugo project 생성

아래의 명령어를 사용하여 새로운 사이트를 생성할 수 있다.

```bash
$ hugo new site [SITENAME]
Congratulations! Your new Hugo site is created in /home/SITENAME.

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/ or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

Visit https://gohugo.io/ for quickstart guide and full documentation.
```

그럼 입력 한 `SITENAME` 으로 디렉토리가 생길 것이고, 그 디렉토리 내의 구조는 아래와 같을 것이다. 

```plain
SITENAME
  - archetypes
    - default.md
  - content
  - data
  - layouts
  - static
  - themes
  - config.toml
```

각 디렉토리에 대한 설명은 아래와 같다.
(참고 - https://gohugo.io/getting-started/directory-structure/)

* archetypes : Hugo에서는 새로은 포스팅을 쓸 때 상단에 title, data 등 글의 정보가 들어가야 한다. hugo new 명령을 사용하여 새로운 글을 생성할 수 있는데, 이 때 생성 될 md 파일의 상단에 나와야 할 title, data 등의 default 양식을 저장한다.
* content : 블로그에서 공개되는 모든 글이 저장되는 경로이다.
* data : Hugo를 사용 한 웹사이트가 생성될 때 사용되는 설정 파일이 저장되는 경로이다. 그런데 내 경우 config.toml에 모든 설정을 저장했기 때문에 data 디렉토리는 따로 사용하지 않았다.
* layouts : 페이지를 보여주기 위한 html 파일이 저장되는 경로이다.
* static : 페이지를 보여줄 때 사용하는 css, js 등의 파일이 저장되는 파일이다.
* themes : 이 디렉토리는 Jekyll과 같이 이미 생성 된 테마를 가져 와 사용할 때 사용하는 디렉토리이다. 테마는 https://gohugo.io/contribute/themes/ 에서 확인할 수 있다.
* config.toml : 블로그를 생성하고 사용하기 위한 설정 파일들이 저장된다.

그런데 디렉토리 구조를 보면 이상한 부분이 있을 것이다.
페이지를 보여주기 위한 html과 css 등의 파일은 `layouts`, `statics` 디렉토리에 저장되는데 `themes`라는 디렉토리가 또 있다.

만약 맘에 드는 테마를 찾아 블로그에 적용 할 경우 해당 코드는 `themes`에 저장된다.
그런데 이 때 일부를 수정하고 싶더라도 `themes` 내의 코드는 누군가가 만든 코드를 가져 온 것이기 때문에 직접 수정하기는 어려움이 있다.
이 때 `layouts`, `statics` 디렉토리를 사용한다. 
`Hugo`에서 사이트를 생성할 때 참조하는 우선순위가 `layouts/statics` > `themes` 이기 때문에, 만약 일부 디자인을 수정하고 싶을 경우 `layouts`나 `statics`에서 코드를 수정하면 된다.

나는 새로운 테마를 생성 할 것이기 때문에, 만약 이미 만들어져 있는 테마를 사용하고 싶을 경우 위의 테마 목록 사이트에서 마음에 드는 테마를 선택하고, 각 테마 별로 적혀있는 사용법을 참고하면 된다.

#### 3. Hugo Project 실행하기

테마를 생성하기 전에, 테마를 수정하기 위해 블로그 내 코드의 수정사항을 실시간으로 확인해야 하기 때문에 `Hugo Project`를 실행하는 방법을 먼저 설명한다.

1번에서 설명한 바와 같이 로컬에서 `hugo`를 설치했을 경우, 아래의 명령어를 통해 생성 한 프로젝트를 실행할 수 있다.

```bash
$ hugo server --theme=THEME_NAME --baseURL "http://IP_ADDRESS" --bind "0.0.0.0"
                   | EN   
-------------------+------
  Pages            | 337  
  Paginator pages  |  39  
  Non-page files   | 494  
  Static files     |   1  
  Processed images |   0  
  Aliases          | 100  
  Sitemaps         |   1  
  Cleaned          |   0  

Built in 546 ms
Watching for changes in /home/mingzzi/blog/{archetypes,content,data,themes}
Watching for config changes in /home/mingzzi/blog/config.toml
Environment: "development"
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://ID_ADDRESS:1313/ (bind address 0.0.0.0)
Press Ctrl+C to stop
```

그럼 위와 같이 `Build 완료` 안내가 나오며, 안내 문구 내 주소로 접속 시 블로그를 확인할 수 있다. (기본적으로 1313 포트 사용)
또한 코드 변경 시 재실행하지 않아도 바로 반영된다.

#### 4. 새로운 테마 생성

새로운 테마를 생성하는 명령어 또한 매우 간단하다.
앞서 생성 한 새로운 사이트의 디렉토리로 들어가 아래의 명령어를 입력한다.

```bash
$ hugo new theme [THEMENAME]
Creating theme at /home/mingzzi/blog/themes/TEHEMNAME
```

그럼 `SITENAME/themes` 디렉토리 아래에 입력 한 `THEMENAME`으로 새로운 디렉토리가 생성되어 있을 것이다.
생성 된 테마 디렉토리의 구조는 아래와 같다.

```plain
THEMENAME
    - archetypes
        - default.md
    - layouts
        - 404.html
        - _default
            - baseof.html
            - list.html
            - single.html
        - index.html
        - partials
            - footer.html
            - header.html
            - head.html
    - static
        - css
        - js
    - theme.toml
    - LICENSE
```

디렉토리 구조는 앞서 설명 한 것과 동일하다.
하지만 앞서 설명 한 구조와는 다르게 `layouts` 디렉토리에 `html` 파일들이 생성된 것을 알 수 있는데, 각 파일의 역할은 테마를 만들며 설명하려 한다.

#### 5. Hugo variable 사용법

테마를 만들기 전에 `Hugo variable`의 간단한 사용 법을 정리한다.
기본적으로 변수는 `{{ .변수명 }}`의 형태로 사용된다.
내가 블로그를 만들며 사용한 변수들 기준으로 설명하며, 더 다양한 변수는 [여기](https://gohugo.io/variables/)에서 확인할 수 있다.

변수에 대해 정리하기 전에 `사이트(Site)`와 `페이지(Page)`의 개념에 대해 정의해야 할 것 같다.
사이트란 Hugo를 통해 블로그를 생성했을 경우 그 블로그 전체를 의미하고, 페이지는 각 글 하나하나 혹은 html로 정의 된 페이지 하나하나를 의미한다.
내 블로그에서 `https://mingzz1.github.io 라는 블로그`는 `사이트`이고, 각 `블로그 포스팅 혹은 Archive, Tag, About 등의 메뉴`들은 `페이지`이다.

##### 1) .Site variable
제목과 같이 전체 사이트를 대상으로 적용되는 변수이다.
가장 많이 사용하는 변수를 몇 가지 정리 해 보려 한다.

* .Site.BaseURL : config.toml 내에서 정의 한 baseURL을 불러오는 변수이다. css 파일 등 리소스를 불러올 때 사용하게 된다. 아래 예시를 보면 baseURL을 "https://mingzz1.github.io/" 라고 선언 해 두었는데, .Site.BaseURL 변수를 사용 할 경우 "https://mingzz1.github.io/"를 의미한다.
* .Site.Pages : 날짜 순으로 정렬 된 content 내 모든 페이지의 목록이다. 이 때, "모든 페이지"에는 regular page, sections, taxonomies 등 모두 포함한다. 즉, 아래에서 설명하겠지만 _index.md 파일이라는게 존재하는데 이 파일까지 포함 된 변수이다. 단순히 내가 작성 한 포스팅의 목록을 보여주고 싶다면 아래의 .Site.RegularPages를 사용해야 한다.
* .Site.RegularPages : _index.md를 제외 한 순수 포스팅의 목록이다. 내 경우 블로그 메인에 들어왔을 때, 혹은 Archive 메뉴에 접속 했을 때 블로그 내 모든 포스팅 목록이 나타나는데, 이 때 .Site.RegularPages 변수를 사용했다.
* .Site.Title : 현재 사이트의 title, 즉 config.toml내 정의 한 title을 의미한다. 아래 예시의 경우 "mingzzi's blog" 라고 선언되어 있기 때문에 .Site.Title 변수를 사용 할 경우 "mingzzi's blog"를 의미한다.
* .Site.Params.PARAMNAME : config.toml내 정의 한 커스텀 변수를 불러올 때 사용한다. config.toml 내에서 변수를 선언하는 방법은 아래와 같다. 아래의 config.toml을 보면 [params] 아래에 favicon이 선언되어 있는데, 이 경우 html 내에서 해당 변수를 불러오고 싶다면 .Site.Params.favicon 으로 사용하면 된다.

```toml
# config.toml
baseURL = "https://mingzz1.github.io/"
languageCode = "ko-kr"
title = "mingzzi's blog"
theme = "triplemode"

[params]
favicon = "img/favicon.ico"
```

##### 2) .Page variable
다음은 페이지에서 사용하는 변수이다.
각 페이지에서 사용하는 변수이기 때문에 주로 Markdown 내 정의 한 내용들을 불러올 때 주로 사용한다.

* .Page.Title : 해당 페이지의 title을 출력하는 변수이다. 아래 예시 파일을 기준으로 보면 .Page.Title 변수를 사용 할 경우 "테스트 페이지"가 출력 될 것이다.
* .Page.Date : 페이지 내 date의 값을 출력하는 변수이다. 아래 예시 파일을 기준으로 보면 date의 값인 2021-08-02가 출력된다.
* .Page.Content : ---를 기준으로 묶인 설정 아래에 있는 페이지의 컨텐츠를 나타내는 변수이다. 블로그 글이라고 생각 한다면 포스팅의 본문이 되며, 아래 예시 파일을 기준으로 보면 "페이지의 내용입니다."가 출력된다.

```html
<!-- example.md !-->
---
title: "테스트 페이지"
date: 2021-08-02
draft: true
---

페이지의 내용입니다.
```

내 경우, 변수는 이정도만 사용했으며, 그때그때 필요 한 변수도 추가로 생성해서 사용했다.

---

원래 한편에 다 넣으려고 했는데 설명을 상세하게 쓰다보니 글이 너무 길어져 다음 편으로 나눠야 할 것 같다.
다음 글에서는 내가 원하는 테마를 만들기 위해 `themes` 폴더 내 `html`의 역할, `scss` 사용법을 정리하고, 기타 설정을 반영하는 방법을 설명하려 한다.

[]()