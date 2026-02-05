---
title: "[Jekyll] Syntax Highlighting with rouge"
date: 2020-06-16
draft: false
category: [development]
subcategories: [github-blog]
tags: [Jekyll, Github Blog]
---

내 블로그는 원래 Jekyll을 사용하고 있었고, Syntax Highlighting을 위해 rouge를 사용하고 있다.
기존에 사용하던 검은 바탕의 `Syntax Highlighting`이 답답하고 가독성이 떨어져 보여 바꾸고 싶어 찾던 중 생각보다 자료가 별로 없어 따로 정리 해 두게 되었다.  

<!--more-->

`rouge`는 `ruby` 기반이기 때문에 아래의 명령어를 통해 설치할 수 있다.  

```sh
$ gem install rouge
```

내 경우, 예전에 아래 포스팅을 쓸 때 로컬에 블로그를 세팅해 둔 가상머신에 이미 설치가 되어 있는 상태였다.  

> [[Jekyll] Local에 Github Blog 세팅하기](https://mingzz1.github.io/development/github%20blog/2020/01/06/jekyll-local_blog_setting.html)

만약 `gem`이 설치되어 있지 않다면 위의 글을 참고해 설치하면 된다.
`rouge`가 설치되었다면 어떤 스타일을 사용 가능한지 확인을 해 보아야 한다.
확인하는 명령어는 `rougify help style`이다.  

```sh
$ rougify help style
usage: rougify style [<theme-name>] [<options>]

Print CSS styles for the given theme.  Extra options are
passed to the theme. To select a mode (light/dark) for the
theme, append '.light' or '.dark' to the <theme-name>
respectively. Theme defaults to thankful_eyes.

options:
  --scope     (default: .highlight) a css selector to scope by
  --tex       (default: false) render as TeX
  --tex-prefix (default: RG) a command prefix for TeX
              implies --tex if specified

available themes:
  base16, base16.dark, base16.light, base16.monokai, base16.monokai.dark, base16.monokai.light, base16.solarized, base16.solarized.dark, base16.solarized.light, bw, colorful, github, gruvbox, gruvbox.dark, gruvbox.light, igorpro, magritte, molokai, monokai, monokai.sublime, pastie, thankful_eyes, tulip
```

내가 테스트한 현재 기준으로는 `23개`의 테마를 사용할 수 있었다.
사용 가능한 테마는 명령어 실행 결과 하단에 나온다.  

```sh
available themes:
  base16, base16.dark, base16.light, base16.monokai, base16.monokai.dark, base16.monokai.light, base16.solarized, base16.solarized.dark, base16.solarized.light, bw, colorful, github, gruvbox, gruvbox.dark, gruvbox.light, igorpro, magritte, molokai, monokai, monokai.sublime, pastie, thankful_eyes, tulip
```

해당 테마를 적용하기 위해서는 아래 명령어를 사용하면 된다.  

```sh
$ rougify style [Theme_Name] > [CSS_Source_Path/Filename]
```

`23개`의 테마 중 마음에 드는 테마를 고른 후 `rougify style 테마명`을 하면 해당 테마의 `css` 코드가 출력되는데, 이를 syntax를 적용하는 css 파일에 넣어주면 된다.
내 경우 `jekyll` 테마에 `_syntax.scss`라는 파일이 존재하고 있어 해당 파일에 마음에 드는 테마인 `github`의 css code를 넣어 주었다.    

```sh
$ rougify style github > _sass/external/_syntax.scss
```

테마가 어떤지 확인 해 보고 싶은데 내가 못찾은 건지, 테마를 확인할 수 있는 곳이 없었다.
그래서 나는 하나하나 다 테스트 해 봤다 ㅠㅠ
다음번에는 이런 수고로움을 덜기 위해 각 테마 별로 샘플 코드를 구현 해 캡쳐를 해 두었다.
마음에 드는 구성을 찾아 사용하면 될 것 같다.  

> base16  

![](/images/development/blog/rougify-theme/base16.png)

> base16.dark  

![](/images/development/blog/rougify-theme/base16.dark.png)

> base16.light  

![](/images/development/blog/rougify-theme/base16.light.png)

> base16.monokai  

![](/images/development/blog/rougify-theme/base16.monokai.png)

> base16.monokai.dark  

![](/images/development/blog/rougify-theme/base16.monokai.dark.png)

> base16.monokai.light  

![](/images/development/blog/rougify-theme/base16.monokai.light.png)

> base16.solarized  

![](/images/development/blog/rougify-theme/base16.solarized.png)

> base16.solarized  

![](/images/development/blog/rougify-theme/base16.solarized.dark.png)

> base16.solarized  

![](/images/development/blog/rougify-theme/base16.solarized.light.png)

> bw  

![](/images/development/blog/rougify-theme/bw.png)

> colorful  

![](/images/development/blog/rougify-theme/colorful.png)

> github  

![](/images/development/blog/rougify-theme/github.png)

> gruvbox  

![](/images/development/blog/rougify-theme/gruvbox.png)

> gruvbox.dark  

![](/images/development/blog/rougify-theme/gruvbox.dark.png)

> gruvbox.light  

![](/images/development/blog/rougify-theme/gruvbox.light.png)

> igorpro  

![](/images/development/blog/rougify-theme/igorpro.png)

> magritte  

![](/images/development/blog/rougify-theme/magritte.png)

> molokai  

![](/images/development/blog/rougify-theme/molokai.png)

> monokai  

![](/images/development/blog/rougify-theme/monokai.png)

> monokai.sublime  

![](/images/development/blog/rougify-theme/monokai.sublime.png)

> pastie  

![](/images/development/blog/rougify-theme/pastie.png)

> thankful_eyes  

![](/images/development/blog/rougify-theme/thankful_eyes.png)

> tulip  

![](/images/development/blog/rougify-theme/tulip.png)
