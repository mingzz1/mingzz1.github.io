---
title: "[Golang + GRPC] Golang + GRPC + React로 웹서비스 구현하기 - 환경구축 편"
date: 2021-04-18
draft: true
category: [development]
subcategories: [web]
tags: [Golang, GRPC, React]
---


예전에 동생과 함께 전화영어 수업 관리용 웹사이트를 구현한 적 있다.
어차피 동생과 나만 쓰기 때문에 그때는 `PHP`로 구현했는데, 동생도 프론트엔드 개발자로, 나도 회사에서 살짝이나마 개발을 하게 되면서 넘나 허접(...)해 보이기도 하고 CSS도 다 깨져서 공부할 겸 새로 개발을 해보기로 했다.  

<!--more-->

사용할 기술 스택은 다음과 같다.  

```
* Golang
* GRPC
* React
```

내가 백엔드를 담당하고, 동생이 프론트엔드 쪽을 개발하게 되어 블로그에는 주로 백엔드 위주로 올라오게 될 것 같다.
GRPC는 최근 다양한 기업들에서 사용하고 있는 기술이며 나도 회사에서 사용하고 있어 알게되었다.
`GRPC가 무엇인가`는 다른 기업 블로그에서 소개해 둔 글이 더 자세하기 때문에 내가 따로 설명하진 않으려 한다.  

[위키백과](https://ko.wikipedia.org/wiki/GRPC)
[뱅크샐러드 블로그](https://blog.banksalad.com/tech/production-ready-grpc-in-golang/?gclid=CjwKCAjwjuqDBhAGEiwAdX2cj5YG4QawRLt7uZzy6fjjLJB1h8SEeTtGWHtGwssr-9MbV624jIvNHBoCE0UQAvD_BwE)
[라인 블로그](https://engineering.linecorp.com/ko/blog/combining-slackbots-into-one-with-grpc/)
[NHN 블로그](https://meetup.toast.com/posts/261)
[당근마켓 블로그](https://medium.com/daangn/%EC%95%88%EC%8B%AC%EB%B2%88%ED%98%B8-%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4-%EA%B0%9C%EB%B0%9C%ED%95%98%EA%B8%B0-fb1a8817b059)
[버즈빌 블로그](https://www.mobiinside.co.kr/2019/09/26/buzzvil-grpc/)

이번 글에서는 먼저 환경 구축만 해보기로 한다.  

### 1. go 설치하기  
나는 Windows를 사용하기 때문에 Windows를 기준으로 설명한다.
먼저 [http://golang.org/dl](http://golang.org/dl)에 접속하여 Go를 다운로드 한다.
다운로드가 완료되면 `go env` 혹은 `go version`을 통해 설치 여부를 확인할 수 있다.

```bash
> go version
go version go1.16.3 windows/amd64
```

내 경우 Github에서 코드를 관리하기 때문에 Git Repository 안에서 작업을 하고있다.
Git Repository를 clone 한 후, 아래 명령어로 Go 모듈을 생성한다.

```bash
> go mod init github.com/USERNAME/PROJECT_NAME
```

그럼 아래와 같은 `go.mod` 파일이 생성된다.  

```go
module MODULE_NAME

go 1.16
```

### 2. GRPC 모듈 import 하기  

이제 `GRPC`를 사용하기 위해 `GRPC` 모듈을 `import` 해야 한다.  

```bash
> go get -u google.golang.org/grpc
go: downloading google.golang.org/grpc v1.37.0
go: downloading golang.org/x/net v0.0.0-20190311183353-d8887717615a
go: downloading github.com/golang/protobuf v1.4.2
go: downloading golang.org/x/sys v0.0.0-20190215142949-d0b11bdaac8a
go: downloading google.golang.org/genproto v0.0.0-20200526211855-cb27e3aa2013
go: downloading github.com/golang/protobuf v1.5.2
go: downloading google.golang.org/protobuf v1.25.0
go: downloading golang.org/x/net v0.0.0-20210415231046-e915ea6b2b7d
go: downloading golang.org/x/sys v0.0.0-20210415045647-66c3f260301c
go: downloading google.golang.org/protobuf v1.26.0
go: downloading golang.org/x/text v0.3.0
go: downloading golang.org/x/text v0.3.6
go: downloading google.golang.org/genproto v0.0.0-20210416161957-9910b6c460de
go get: added golang.org/x/net v0.0.0-20210415231046-e915ea6b2b7d
go get: added golang.org/x/sys v0.0.0-20210415045647-66c3f260301c
go get: added google.golang.org/genproto v0.0.0-20210416161957-9910b6c460de
go get: added google.golang.org/grpc v1.37.0

```

그럼 `go.mod` 파일에 아래 내용들이 추가된다.  

```go
module MODULE_NAME

go 1.16

require (
	golang.org/x/net v0.0.0-20210415231046-e915ea6b2b7d // indirect
	golang.org/x/sys v0.0.0-20210415045647-66c3f260301c // indirect
	google.golang.org/genproto v0.0.0-20210416161957-9910b6c460de // indirect
	google.golang.org/grpc v1.37.0 // indirect
)
```