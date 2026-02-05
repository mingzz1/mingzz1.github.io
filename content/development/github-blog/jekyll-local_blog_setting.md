---
title: "[Jekyll] Local에 Github Blog 세팅하기"
date: 2020-01-06
draft: false
category: [development]
subcategories: [github-blog]
tags: [Jekyll, Github Blog]
---

Github 블로그를 운영 중에 게시글이 많아지며 블로그 리뉴얼의 필요성을 느꼈다.
테마를 싹 바꾸는 대대적인 작업이기 때문에 먼저 로컬에서 작업 후, 완성본을 Github에 push 하기로 했다.  

<!--more-->

나는 Ubuntu 16.04 Server version에 세팅을 했다.
설치를 해야하는 항목은 다음과 같다.  

* Nodejs
* RubyGems
* Ruby
* gcc / g++ / make

각각의 설치 방법은 다음과 같다.  

```sh
$ apt-get install -y nodejs
$ apt-get install -y rubygems
$ apt-get install -y ruby
$ apt-get install -y ruby-dev
$ apt-get install -y gcc
$ apt-get install -y g++
$ apt-get install -y make
```

이제 `Jekyll`을 설치해야 한다. 만약 `gem install jekyll`을 하면 `jekyll 4.0.0`이 설치되는데, 이 경우 해결해야 할 오류가 너무 많다. 그래서 나는 그냥 `jekyll 3.8.5`를 설치했다.  

```sh
$ gem install jekyll -v 3.8.5
$ gem install bundler
```

여기까지 하면 모든 준비가 완료된다.
이제 내 Github에서 내 블로그 Repository를 Clone 한다.  

```sh
$ git clone https://github.com/mingzz1/mingzz1.github.io.git
$ cd mingzz1.github.io
```

Clone을 했으면 필요한 플러그인을 설치하고 serve를 해야 한다. (내 경우, Gemfile을 다 지워두었어서 원래 테마의 Github에서 Gemfile과 Gemfile.lock를 가져왔다.)  

```sh
$ bundle install
```

만약 `bundle install`을 하는 동안 오류가 난다면, 해당 오류를 잘 읽어보면 된다.
내 경우 아래와 같은 오류가 계속해서 발생했다.  

```sh
jekyll-4.0.0 requires ruby version >= 2.4.0, which is incompatible with the current version, ruby 2.3.1p112
```

이 경우, ruby 버전을 2.4 이상을 설치하거나 Gemfile 혹은 Gemfile.lock에서 jekyll 버전을 낮추면 된다.
이 후 만약 아래의 오류가 발생한다면 `bundle update jekyll`을 하면 된다.  

```sh
Downloading jekyll-3.8.3 revealed dependencies not in the API or the lockfile (i18n (~>
0.7), jekyll-sass-converter (~> 1.0), kramdown (~> 1.14), rouge (< 4, >= 1.7)).
Either installing with `--full-index` or running `bundle update jekyll` should fix the
problem.
```

`bundle install`이 완료되었다면 이제 serve를 하면 된다.
serve 시, 나는 Server version의 Ubuntu를 사용했기 때문에 호스트에서 접속해야 해서 따로 호스팅 주소를 넣어 주었다.
이 주소는 해당 가상머신 혹은 서버의 IP 주소를 넣어주면 된다.  

```sh
$ bundle exec jekyll serve -H 192.168.222.144
```

웹 브라우저로 `http://192.168.222.144:4000`에 접속하면 내 블로그가 떠 있는 것을 확인할 수 있다.  

![](/images/development/blog/local_set/local_set_01.png)
