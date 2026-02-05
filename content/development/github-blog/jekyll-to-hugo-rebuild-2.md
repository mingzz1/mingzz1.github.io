---
title: "[Hugo] Jekyll to Hugo 블로그 이전기 2"
date: 2021-08-03
draft: false
category: [development]
subcategories: [github-blog]
tags: [Hugo, Github Blog]
---

이 전 포스팅에 이어서 `Jekyll`에서 `Hugo`로 블로그를 이전 한 내용을 기록하려 한다. 
이번에는 앞서 예고했듯 내가 원하는 테마를 만들기 위해 `themes` 폴더 내 `html`의 역할, `scss` 사용법을 정리하고, 기타 설정을 반영하는 방법을 설명한다.

<!--more-->

#### 6. _index.md

테마 수정 방법을 설명하기 전에 `_index.md`를 먼저 설명해야 할 것 같다.
`_index.md`는 각 폴더의 정보를 가지고 있는 파일로, `content` 폴더 내 모든 폴더에 생성해야 한다.

만약 각 폴더 내에서 불러와야 할 정보가 없을 경우에는 빈 파일로 생성해도 무관하지만, 사용해야 하는 정보가 있는 경우에는 아래와 같이 필요한 정보를 적어주면 된다.
내 경우에는 `각 폴더 = 카테고리`의 형태이기 때문에 카테고리 별 포스팅 정보를 출력 해 주어야 했다.
때문에 각 폴더 내에 `title`을 선언하고, 카테고리 명을 적어주었다.

```markdown
---
title: "Wargame/Wargame.kr"
---
```

`_index.md`가 없을 경우, 해당 폴더 내 글 목록을 리스팅 하는 것도 불가능하기 때문에 빈 파일로라도 생성 해 주는 것이 좋다.

#### 7. themes 구조

앞의 글에서 `hugo new theme THEMENAME` 명령어로 테마를 생성하면 아래와 같은 구조가 생성된다고 설명했다.  

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

각 폴더 및 파일의 설명은 아래와 같다.

##### 1) archetypes 폴더
앞서 설명했듯이 생성 될 md 파일의 상단에 나와야 할 title, data 등의 default 양식을 저장한다.
default.md 에 양식을 저장 해 두면 새로운 글 생성 시 해당 양식에 맞추어 글이 생성된다.
내 경우 아래와 같이 지정 해 두고 사용하고 있다.

```plain
---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
category: []
subcategories: []
tags: []
---
```

때문에 `hugo new PATH`를 할 경우 위의 양식과 같이 지정한 경로에 `markdown` 파일이 생성된다.

##### 2) layouts 폴더
`layouts` 폴더에는 페이지를 표시하기 위한 `html` 파일들이 저장된다.
앞의 글에서 설명했듯 만약 블로그 프로젝트 내의 `layouts` 폴더에 파일이 있을 경우 그 폴더 내의 `html` 파일이 우선순위가 높다.
하지만 나는 내가 테마를 따로 생성하므로, 이 폴더 내의 `html` 파일만 수정 할 예정이다.
만약 타인이 생성 한 테마를 가져다 쓰는 경우, 직접적으로 수정하기 어려울 수 있기 때문에 그 때는 블로그 프로젝트 내의 `layout` 폴더를 수정하면 된다.

각 파일 별 설명은 아래와 같다.
[Hugo 공식 페이지의 설명](https://gohugo.io/templates/base/)에 나온 코드를 기준으로 설명한다.
내 코드가 궁금하다면 [Github Repository](https://github.com/mingzz1/blog)에서 확인하면 된다.

* layouts/_default/baseof.html  
```html
<!-- layouts/_default/baseof.html -->
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>{{ block "title" . }}
      <!-- Blocks may include default content. -->
      {{ .Site.Title }}
    {{ end }}</title>
  </head>
  <body>
    <!-- Code that all your templates share, like a header -->
    {{ block "main" . }}
      <!-- The part of the page that begins to differ between templates -->
    {{ end }}
    {{ block "footer" . }}
    <!-- More shared code, perhaps a footer but that can be overridden if need be in  -->
    {{ end }}
  </body>
</html>
```
블로그 전체 페이지의 `틀` 역할을 한다고 보면 쉬울 것 같다.
모든 페이지를 표현하는 기본 페이지이며, 코드 중 `{{ block "title" . }}`, `{{ block "main" . }}` 혹은 `{{ block "footer" .}}` 부분이 있는데, 이 부분에 `title`, `main` 혹은 `footer`라고 정의 한 `html` 코드가 `include` 된다.
각 `block`들은 `end`를 사용하여 닫아주어야 한다.

* layouts/_default/list.html  

```html
<!-- layouts/_default/list.html -->
{{ define "main" }}
  <h1>Posts</h1>
  {{ range .Pages }}
    <article>
      <h2>{{ .Title }}</h2>
      {{ .Content }}
    </article>
  {{ end }}
{{ end }}
```
블로그에서 글의 목록을 보여주는 페이지이다.
`{{ define "main" }}` 코드는 해당 코드블럭 내의 코드들을 `main`이라고 정의한다는 의미이며, 앞서 `baseof.html`에서 설명한 `{{ block "main" . }}` 안에 들어가게 된다.
또한 `{{ range .Pages }}`라는 반복문을 사용하여 사이트 내 각 페이지의 정보를 출력하고 있으며, `{{ .Title }}` 에서는 해당 페이지의 제목을, `{{ .Content }}`에서는 해당 페이지의 본문을 출력한다.
`define`과 `range` 또한 `end`를 사용하여 닫아주어야 한다.

* layouts/_default/single.html  
```html
<!-- layouts/_default/single.html -->
{{ define "title" }}
  <!-- This will override the default value set in baseof.html; i.e., "{{.Site.Title}}" in the original example-->
  {{ .Title }} &ndash; {{ .Site.Title }}
{{ end }}
{{ define "main" }}
  <h1>{{ .Title }}</h1>
  {{ .Content }}
{{ end }}
```
각 글을 눌렀을 때 글을 보여주기 위한 `html` 코드이다.
여기서 `{{ .Title }}` 및 `{{ .Content }}`는 해당 페이지의 md 파일에서 선언 한 title과 본문을 출력하게 된다.

* layouts/index.html
```html
<!-- layouts/index.html -->
{{ define "main" }}
    <main aria-role="main">
      <header class="homepage-header">
        <h1>{{.Title}}</h1>
        {{ with .Params.subtitle }}
        <span class="subtitle">{{.}}</span>
        {{ end }}
      </header>
      <div class="homepage-content">
        <!-- Note that the content for index.html, as a sort of list page, will pull from content/_index.md -->
        {{.Content}}
      </div>
      <div>
        {{ range first 10 .Site.RegularPages }}
            {{ .Render "summary"}}
        {{ end }}
      </div>
    </main>
{{ end }}
```
블로그에 가장 먼저 들어왔을 때 보이는 메인 페이지이다.
예시 코드의 경우 `content/_index.md` 파일 내 선언 한 title과 본문을 출력 한 후 블로그 내 포스팅 중 10개의 간략한 용약을 출력하는 코드이다.

* layouts/404.html
```html
<!-- /layouts/404.html -->
{{ define "main"}}
    <main id="main">
      <div>
       <h1 id="title"><a href="{{ "/" | relURL }}">Go Home</a></h1>
      </div>
    </main>
{{ end }}
```

이름에서도 알 수 있듯이 페이지를 찾을 수 없을 때 보이는 페이지이다.

* layouts/partials/*.html

`partials` 내 `html` 코드들은 다른 html 코드에서 `include` 하여 사용하는 코드이다.
때문에 주로 `header`, `head` 혹은 `footer`를 `partials` 폴더 내에 생성한다.

`partials`내 선언 한 코드를 다른 html 코드에서 불러오고 싶을 경우 `{{ partial "FILENAME" . }}` 으로 사용하면 된다.
즉, 만약 `partials/header.html`이 있고 이 `header.html`을 html 코드에서 불러오고 싶을 경우 코드는 아래와 같다.

```html
<html>
    <head>
        {{ partial "header" . }}
    </head>
    <body>
        <!-- Contents !-->
    </body>
</html>
```

테마를 새로 생성 한 경우 `themes` 폴더 내 `html` 코드들을 위의 설명과 같이 목적에 따라 자유롭게 수정하여 사용하면 될 것 같다.

#### 8. scss 사용법

마지막으로 `scss` 사용법이다.
내가 `Jekyll`을 사용할 때 모든 `css` 코드들은 `scss` 코드로 구현되어 있어 잠깐 써 보았는데 `css`에 비해서 훨씬 사용하기가 용이한 것 같아 이번 블로그 리뉴얼때도 사용했다.

`scss`는 `css 전처리기`라고 불리며, css 좀 더 효율적으로 작성할 수 있게 도와준다.
컴파일이 필요하다는 것은 단점이라고 볼 수 있겠지만, 변수를 사용할 수 있고 `@mixin`을 통해 재사용이 필요한 내용들을 쉽게 불러와 사용할 수 있다.

컴파일이 필요하기 때문에 `scss`를 `Hugo` 내 기본 테마 틀에서는 사용이 불가능하고 `themes` 폴더 아래에 `assets/sass` 폴더를 새로 생성해야 한다.
폴더 생성 후에는 `themes/assets/sass` 아래에 `.scss`을 구현하고, 이를 불러오기 위해서는 `<head>` 태그 아래에 아래와 같이 구현하면 된다.

```html
<head>
    {{ $style := resources.Get "sass/main.scss" | toCSS | minify | fingerprint }}
    <link rel="stylesheet" href="{{ $style.Permalink }}">
</head>
```

`resource.Get` 옆에는 `themes/assets/sass` 폴더 아래에서 `scss` 파일들의 메인이 되는 파일의 경로를 적어주면 된다.
이렇게 선언 해 두면 블로그 빌드 시 `scss`도 함께 컴파일 되어 반영된다.
블로그 빌드 방법은 아래에서 설명한다.

#### 9. Category, Tag 적용하기

내가 블로그 이전을 고려할 때 중요하게 생각했던 조건이 `Tag`와 `Category` 구조이다.
두 기능 모두 이 전에 `Jekyll`에서 사용하고 있었기 때문에 없으면 글을 찾기 매우 불편할 것 같아 해당 기능은 꼭 넣으려고 했다.

Tag와 Category를 사용하기 위해서는 먼저 `config.toml`에 아래와 같이 선언을 해 주어야 한다.

```plain
[taxonomies]
    category = "categories"
    tag = "tags"
    subcategory = "subcategories"
```

각각의 사용법은 아래에서 설명한다.

##### 1) Category

먼저 카테고리 구조이다.
내 경우에는 `content` 폴더 아래에 카테고리 별로 폴더를 생성하여 `같은 폴더 내 존재하는 포스팅 = 같은 카테고리로 분류` 했다.
그 후, 각 포스팅의 상단에 아래와 같이 정보를 적어 주었다.

```plain
---
title: "블로그 제목"
date: 2021-08-04
draft: false
category: [category]
subcategories: [subcategory]
tags: [tag1], tag2]
---
```

여기서 `category`에는 `메인 카테고리 명`, `subcategories`에는 `서브 카테고리 명`을 적었다.
예를 들어서 어떤 포스팅의 경로가 `content/category1/category_a/test_posting.html`일 경우, `category`는 `category1`, `subcategories`는 `category_a`가 되는 것이다.

이 후 각 카테고리 별로 메뉴를 만들어 메뉴 클릭 시 해당 카테고리 내 포스팅을 보여주기 위해서 코드를 구현했다.
먼저 메뉴 부분은 아래와 같다.

```html
<li><a>category1</a>
    <ul>
        <li><a href="{{ .Site.BaseURL }}/category1/category_a">Category A</a></li>
        <li><a href="{{ .Site.BaseURL }}/category1/category_b">Category B</a></li>
        <li><a href="{{ .Site.BaseURL }}/category1/category_c">Category C</a></li>
        <li><a href="{{ .Site.BaseURL }}/category1/category_d">Category D</a></li>
    </ul>
    <span class="down"></span>
</li>
```

카테고리 클릭 시에 이동하여 카테고리 내 포스팅을 보여주는 코드는 아래와 같다.

```html
<div class="home">
    <div class="posts">
        <div>
            <h3> {{ .Page.Title }} <span>{{ len .Pages }}</span></h3>
            <ul class="post-box">
                {{ range .Pages }}
                <li class="post-teaser">
                    <a href="{{.Permalink}}">
                        <div>
                            <h4>{{.Title}}</h4>
                            <div class="post-info">
                                <p class="meta">
                                    {{.Date.Format "2006-01-02"}}
                                </p>
                                <div class="excerpt">
                                    {{ .Summary }}
                                </div>
                            </div>
                        </div>
                    </a>
                </li>
                {{ end }}
            </ul>
        </div>
    </div>
</div>
```

이 때, 각 폴더의 아래에는 `_index.md`가 있어야 하며, `.Page.Title`은 `_index.md`에 적은 `title` 값이, `len .Pages`는 해당 폴더 아래의 글의 목록을 의미한다.

##### 2) Tag
태그는 카테고리에 비해 매우 간단하다.
각 포스팅을 작성할 때 상단에 아래와 같은 정보를 적어야 한다고 설명했다.

```plain
---
title: "블로그 제목"
date: 2021-08-04
draft: false
category: [category]
subcategories: [subcategory]
tags: [tag1], tag2]
---
```

태그를 사용하기 위해서는 `tags` 옆에 `[태그명]`의 형태로 각 글에 적어주면 해당 글의 태그는 적용이 된다.
그럼 적용한 태그를 `포스팅 내에서 태그를 표시` 해 주고, `각 태그 별로 글 찾기`가 가능해야 한다.

먼저 포스팅 내에서 태그를 표시하기 위해서는 `html`을 아래와 같이 구현하면 된다.

```html
<footer>
    <div class="tags">
        {{ range (.GetTerms "tags") }}
        <a href="{{ .Site.BaseURL }}/tags#{{ .LinkTitle }}">#{{ .LinkTitle }}</a>
        {{ end }}
    </div>
</footer>
```

나는 `partials` 아래에 `tag-list.html`이라는 파일을 생성해서 위와 같이 구현 해 두었다.

각 태그 별로 글을 찾기 위해서는 `_default` 아래에 `terms.html` 이라는 이름으로 파일을 생성 한 후 아래와 같이 구현하면 된다.

```html
{{ define "main" }}

<div class="feature-image">
    <header>
        <h1 class="title ar-title">
            tags
        </h1>
    </header>
    <section class="post-content">
        {{ range .Site.Taxonomies.tags }}
        <h2 id="{{ .Page.Title }}" class="tag-title">
            #{{ .Page.Title }}
        </h2>

        {{ range .Pages }}
        <div class="tagged-post">
            <h3 class=title>
                <a href="{{ .Permalink }}">
                    {{ .Title }}
                </a>
                <small>
                    {{.Date.Format "2006-01-02"}}
                </small>
            </h3>
        </div>

        {{ end }}
        {{ end }}

    </section>
</div>
{{ end }}
```

위와 같이 구현을 완료한다면 "https://exampleblog.com/tags"로 접속하여 각 태그 목록을 확인할 수 있다.

#### 10. Github Pages로 배포하기

블로그를 완성했다면 이제 완성 된 블로그를 `github.io`의 형태로 배포해야 한다.
`Hugo`는 `Jekyll`과 달리 내가 빌드를 한 후 commit을 해야 하기 때문에 무려 2개의 Repository가 필요하다.

두 개의 Repository는 편의 상 각각 `blog`와 `github.io`라고 명명하려 한다.
`blog` Repository에는 여태까지 수정 한 모든 코드들이 저장되는 Repository 이다.
반면 `github.io`는 빌드 후 실제로 블로그를 뿌려주는 역할, 즉 `github.io`로 접속했을 때 보여지는 코드들이 있는 Repository이다.
때문에 `blog` Repository는 생성 시 이름을 어떻게 해도 상관 없지만, `github.io` Repository는 `GITHUB_ID.github.io`로 만들어야 한다.

내 경우 `blog` Repository의 이름은 [blog](https://github.com/mingzz1/blog)로, `github.io` Repository의 이름은 [mingzz1.github.io](https://github.com/mingzz1/mingzz1.github.io)로 생성했다.

앞서 설명했듯이, 여태까지 작성한 모든 코드는 `blog` Repository에 저장한다.
이 후, `github.io` Repository를 Submodule로 지정해야 한다.
모든 명령어는 Linux를 기준으로 설명한다.

```bash
$ cd blog
$ git init
$ git remote add origin [blog Repository 경로]
$ git submodule add -b [Default branch 명] [github.io Repository 경로] public
```

내 경우에는 `blog Repository`의 주소는 `https://github.com/mingzz1/blog.git`이고, `github.io Repository`의 주소는 `https://github.com/mingzz1/mingzz1.github.io.git`이며, 두 Repository의 Default branch 명은 `main`이다.
때문에 아래와 같이 입력하면 된다.

```bash
$ git init
$ git remote add origin https://github.com/mingzz1/blog.git
$ git submodule add -b main https://github.com/mingzz1/mingzz1.github.io.git public
```

명령어 입력 후에는 빌드 후에 commit을 해야 한다.

```bash
# build
$ cd blog
$ hugo -t THEMENAME

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

Total in 723 ms
# github.io에 commit
$ cd public
$ git add .
$ git commit --m "commit message"
$ git push -u origin main
# blog에 commit
$ cd ..
$ git add .
$ git commit --m "commit message"
$ git push -u origin main
```

위와 같이 build -> github.io에 commit -> blog에 commit 이라는 3단 과정을 거쳐야 비로소 `https://GITHUB_ID.github.io`에서 결과물을 확인할 수 있다.

#### 11. 기타 꿀팁

마지막으로! `Hugo`를 사용하며 발견한 팁~~이라면 팁이고 당연하다면 당연한 이야기~~이다.

1. 각 포스팅 내 md 파일에 date가 있는데, 이 날짜를 현재 날짜보다 미래의 날짜로 적을 경우 해당 날짜가 될 때까지 포스팅이 공개되지 않는다.
2. md 파일 내 draft에 true를 적을 경우 말 그대로 draft 파일이 되어 commit을 하더라도 글이 공개되지 않는다.
3. 각 포스팅의 경로를 내가 설정할 수 있으며, 이 경로는 content 아래의 폴더 별로 다르게 지정할 수 있다. 경로는 permalink라고 부르며 config.toml에서 아래와 같은 형태로 구현할 수 있다. 아래와 같이 구현 할 경우 content/DIRECTORY_NAME 아래의 포스팅들의 주소는 "https://exampleblog.com/2021/08/04/testposting.html"이 될 것이며, content/DIRECTORY_NAME2 아래의 포스팅들의 주소는 "https://exampleblog.com/2021/testposting.html"가 된다. 더 자세한 정보는 [여기](https://gohugo.io/content-management/urls/#permalinks-configuration-example)를 참고하면 되며, content 폴더 아래에 폴더가 없다면 한번에 permalink를 지정하는 것이 가능하나, 나처럼 여러 폴더로 구성되어 있을 경우에는 각 폴더 별로 permalink를 설정 해 주어야 적용된다.

```plain
[permalinks]
  DIRECTORY_NAME = "/:year/:month/:day/:filename.html"
  DIRECTORY_NAME2 = "/:year/:filename.html"
```

4. About 페이지를 생성하기 위해서는 content 폴더 아래에 about.md 파일을 생성하여 내용을 입력하고 테마 폴더의 layouts/_default 아래에 about.html을 생성하여 코드를 구현하면 "https://exampleblog.com/about"에 접속 시 about.md에 적은 내용을 확인할 수 있다.
5. Archive 페이지도 About 페이지와 마찬가지로 content 폴더 아래에 archive.md 파일을 생성하여 내용을 입력하고 테마 폴더의 layouts/_default 아래에 archive.html을 생성하여 코드를 구현하면 "https://exampleblog.com/archive"에 접속 시 구현 한 내용을 확인할 수 있다. 내 경우 archive는 블로그 내 모든 파일을 연도별로 출력하기 때문에 archive.md에는 기본적인 정보만 있으나, archive.html은 아래와 같이 구현하였다.

```html
{{ define "main" }}
<div class="feature-image">
    <header>
        <h1 class="title ar-title">
            {{ .Title }}
        </h1>
    </header>

    <section class="post-content">
        {{ $prev := 3000}}
        {{range .Site.RegularPages}}
        {{if .Date}}
        {{if gt $prev (.Date.Format "2006")}}
        {{ if ne $prev 3000}}
        {{ end }}
        <h2 id='{{ .Date.Format "2006" }}' class="tag-title">
            {{ .Date.Format "2006" }}
        </h2>

        <div class="tagged-post">
            {{end}}
            <h3 class=title>
                <a href="{{ .Permalink }}">
                    {{ .Title }}
                </a>
                <small>
                    {{.Date.Format "2006-01-02"}}
                </small>

            </h3>
            {{ $prev = .Date.Format "2006"}}
            {{end}}
            {{end}}
        </div>

    </section>
</div>
{{ end }}
```

---

이상으로 `Jekyll에서 Hugo로 블로그 이전기`였다.
사실 블로그 리뉴얼을 7월쯤 마무리 했기 때문에 이 포스팅도 예~전에 올라왔어야 하지만...
1편도 사실 7월에 이미 완성 한 상태였는데...
나는 여름에는 굉장히 무기력해지기 때문에 미루고미루고미루다 이제서야 글을 정리해서 올리게 되었다 ㅠㅠ

`Hugo`로 올린 후 만족도는 아직까지는 매우 높은편이다!
일단 블로그를 수정했을 때 반영되는 속도가 매우 빠르다는게 제일 좋다.
앞으로 또 언제 블로그를 리뉴얼 할 지 모르겠지만, 다음번에는 더 멋있게 수정해 보고 싶다. 화이팅!!!