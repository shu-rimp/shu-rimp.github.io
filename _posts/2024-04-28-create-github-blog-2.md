---
title: github.io 블로그 튜토리얼 - (2) 블로그 단장하기
author: shurimp
date: 2024-04-29 00:58:00 +0900
categories: [Blogging, Tutorial]
tags: [blog, git, github, tutorial]
image:
  path: 1234035817765142548/1234035916889133156/devices-mockup.png?ex=662f4501&is=662df381&hm=fda86f2eb73b7f34a7894d7b0fc0323539294d512fc1cc75b6c38c92f3cadf1c
  alt: github blog
---

> **이전 포스팅 바로가기** <br>
> [(1) 시작하기](/posts/create-github-blog-1) <br>
> (2) 블로그 단장하기

---

초기 설정을 끝냈으면, 이제 블로그를 쓸만하게 다듬는 작업이 필요합니다. 이번 포스트에서는 기본적인 config 설정, 색상 바꾸기, 댓글, 조회수 카운터를 설정하는 방법에 대해 작성합니다.

## Config 설정하기

### config.yml 설정
<kbd>_config.yml</kbd> 파일에서 사이트 configuration 설정을 할 수 있습니다. 기본적으로 변경해야하는 것들 위주로만 변경했습니다.

```yaml
lang: ko
timezone: Asia/Seoul # 시간대를 설정합니다.

title: Tools in hand, brain in gear # 메인 타이틀로, 브라우저 탭에 표시됩니다.

tagline: Backend Engineer # 사이드바 프로필 이름 밑에 이탈릭체로 표시됩니다.

description: >- # seo meta 정보에 사용됩니다.(링크 공유 시 하단에도 나옵니다.)
  Shurimp's Tech Blog.

# Fill in the protocol & hostname for your site.
# e.g. 'https://username.github.io', note that it does not end with a '/'.
url: "https://shurimp.com" # 블로그 주소

github:
  username: shu-rimp # change to your github username

# 사용하지 않는 sns는 주석처리 합니다.
# twitter: 
#   username: twitter_username # change to your twitter username

# 사용하지 않는 sns는 주석처리 합니다.
social:
  # Change to your full name.
  # It will be displayed as the default author of the posts and the copyright owner in the Footer
  name: shurimp
  email: soooojip@gmail.com # change to your email address
  links:
    # The first element serves as the copyright owner's link
    # - https://twitter.com/username # change to your twitter homepage
    - https://github.com/shu-rimp # change to your github homepage
    # Uncomment below to add more social links
    # - https://www.facebook.com/username
    # - https://www.linkedin.com/in/username
```
{: file='/_config.yml'}

상단에서 설정할 수 있는 항목들이고, 하단 설정은 밑에서 설명합니다.

### authors.yml 설정
<kbd>_data/authors.yaml</kbd> 파일에 글의 작성자 정보를 저장합니다. 이 파일에 정보를 저장해놓고 포스팅할 때 상단 메타 정보에서 불러올 수 있습니다.

```yaml
shurimp:
  name: shurimp
  url: https://github.com/shu-rimp
```
{: file='/_data/authors.yml'}

![author][author]
_이 부분에 보여집니다._

### contact.yml 설정
![contact][contact]
_사이드바 하단_

사이드바에 원하는 링크를 표시하고 싶다면, <kbd>_data/contact.yml</kbd> 파일에서 보여주고 싶은 옵션만 남기고 주석처리 하면 됩니다.

```yaml
#  The contact options.

- type: github
  icon: "fab fa-github"

# - type: twitter
#   icon: "fa-brands fa-x-twitter"
```
{: file='/_data/contact.yml'}

이외 <kbd>_data/share.yml</kbd>에서는 링크 공유 옵션을 조절할 수 있습니다.

### 프로필 정보 수정
기본으로 블로그 메인 타이틀이 프로필 이름으로 보여지게 되어있는데, 프로필 이름만 따로 설정하고 싶을 경우 <kbd>_includes/sidebar.html</kbd>에서 해당 부분을 직접 편집하면 됩니다. (파일 상단에 위치해 있습니다.)

이탤릭체 밑에 글을 더 추가하고 싶을 때도 하단에 적고 싶은 문구를 추가하면 반영됩니다.

![sidebar][sidebar]

![profile][profile]
_이렇게 보여집니다._

### favicon 설정
[favicon generator](https://www.favicon-generator.org/)에서 간단한 파비콘을 제작할 수 있습니다. 다운로드 받은 zip파일 압축을 해제하고 모든 파일을 <kbd>assets/img/favicons</kbd> 디렉토리 안에 대체합니다.

![favicon][favicon]

## 블로그 색상 변경
기본 디자인이 매우 마음에 들기 때문에 따로 수정은 하고 싶지 않은데, 링크와 태그 호버 색상이 너무 당황스럽게 빨강 파랑이라 거슬려서 변경합니다.

![link hover][link-hover]
_당황스러운 빨강_
![tag hover][tag-hover]
_당황스러운 파랑_

대부분 색상 변경은 <kbd>_sass/colors</kbd> 디렉토리 내에 `.scss` 파일을 수정하면 반영되는데, 이 둘을 설정하는 파일은 이 디렉토리에 없어서 좀 헤맸습니다.

### 링크 호버 색상 변경
<kbd>_sass/addon/module.scss</kbd> 파일에서 `%link-hover` 블록을 수정합니다. `!important` 옵션을 넣어주어야 반영됩니다. 보라색을 좋아하기 때문에 보라색으로 설정했습니다.

```scss
%link-hover {
  color: #7f52ff !important;
  border-bottom: 1px solid #7f52ff;
  text-decoration: none;
}
```

### 포스트 태그 호버 색상 변경
<kbd>_sass/addon/commons.scss</kbd> 파일에서 `post-tag` 블록을 수정합니다. `#panel-wrapper` 블록 내에 위치합니다. border-color의 경우 `!important` 추가해야 반영됩니다.

![post tag][post-tag]


### 기타 색상 변경
이외 색상 변경은 <kbd>_sass/colors</kbd> 디렉토리의 `.scss` 파일을 수정합니다. 거의 링크 색깔만 바꿨는데, 우측 바로가기 메뉴 색상은 `--toc-highlight`에서 설정할 수 있습니다. `-dark.scss`는 다크모드일 때의 색상, `-light.scss`는 라이트모드일 때의 색상으로 두 군데 모두 바꿔주는 것이 좋습니다.

## 댓글 설정하기

사람들이 포스트에 댓글을 달 수 있는 기능을 추가합니다. disqus, utterances, giscus 등이 있는데, 요즘 가장 많이 사용하는 utterances를 사용했습니다. chirpy의 경우, 세 가지 모두 지원하기 때문에 간편하게 설정할 수 있습니다.

그 전에 먼저 이슈와 라벨 설정을 합니다. utterances는 깃허브 이슈를 생성하는 방식으로 댓글을 등록합니다.

1. 깃허브에서 이슈가 활성화되어있는지 체크합니다.
    ![issue][issue]

2. <kbd>Issues</kbd> 탭에서 새 라벨을 생성합니다. 이 과정은 생략해도 되지만 적용해놓으면 다른 이슈들과 구분되어 좀 더 보기 좋습니다.
    ![label][label]

3. _config.yml 파일을 편집합니다.

    ```yaml
    comments:
    # Global switch for the post comment system. Keeping it empty means disabled.
    provider: utterances # [disqus | utterances | giscus]
    # The provider options are as follows:
    utterances:
        repo: shu-rimp/shu-rimp.github.io # <gh-username>/<repo>
        issue_term: title # < url | pathname | title | ...>
        label: "💬comment"
    ```
    {: file='/_config.yml'}

    - `provider`: utterances
    - `repo`: 레포지토리 이름
    - `issue_term`: 이슈를 생성할 때 제목의 형식을 지정합니다. url, pathname, title 중 취향에 맞게 선택하면 됩니다.
    - `label`: 생성한 라벨 명을 입력합니다. **디폴트 파일에는 없는 부분이라 추가해야 합니다.**
    
    > 만약 해당 방식을 사용하지 않고 utterances 사이트에서 직접 스크립트를 생성할 경우 theme이 지정되는데, theme이 정해져있을 경우 라이트/다크모드 전환 시 색상이 바뀌지 않으므로 제거하는 것을 권장합니다.
    {: .prompt-tip}

4. utterances.html 수정

    `label` 필드를 추가했기 때문에, 스크립트에서 이를 불러올 수 있도록 수정합니다.

    <kbd>_includes/comments/utterances.html</kbd>파일 상단 스크립트 태그에 label을 추가합니다. 

    ![label html][label-html]

5. 댓글이 잘 생성되는지 확인합니다.

    ![comment][comment]
    
    ![comment issue][comment-issue]
    _이슈 탭에서는 이렇게 보여집니다._


## 조회수 카운터 설정하기
조회수 카운터 역시 chirpy에서 지원하는 goatcounter를 사용합니다. 이메일 계정 생성이 필요합니다. 

1. goatcounter 회원가입

    [https://www.goatcounter.com/signup](https://www.goatcounter.com/signup) 링크에서 회원가입 합니다. 가입 후 이메일인증 1회가 필요합니다. Code에는 중복없는 문자를 입력합니다.

    ![gct signup][gct-signup]

2. settings

    우측 상단 <kbd>Settings</kbd> 메뉴에서 다음 정보를 입력합니다.

    ![gct settings][gct-settings]

    `Code`에는 회원가입 당시 입력한 Code를 입력하고, Custom Domain이 있을 경우 원하는 서브도메인을 입력합니다.

3. _config.yml 파일을 편집합니다.

    ```yaml
    # Web Analytics Settings
    analytics:
    google:
        id: # fill in your Google Analytics ID
    goatcounter:
        id: shu
    umami:
        id: # fill in your Umami ID
        domain: # fill in your Umami domain
    matomo:
        id: # fill in your Matomo ID
        domain: # fill in your Matomo domain

    # Pageviews settings
    pageviews:
    provider: goatcounter # now only supports 'goatcounter'
    ```
    {: file='/_config.yml'}

    goatcounter id에 code, provider에 goatcounter를 입력합니다.

4. 데이터를 확인합니다.

    조회수 데이터는 배포한 사이트로 연결했기 때문에 배포 후에 확인할 수 있습니다. 변경사항을 push하고 블로그 접속 후, goatcounter 대시보드로 이동하면 방문 데이터를 확인할 수 있습니다.

    ![gct dashboard][gct-dashboard]

    ![pageview][pageview]
    _블로그에도 이렇게 조회수가 표시됩니다._

    > 데이터가 뜨지 않을 경우, 브라우저의 adblock 확장프로그램이 원인일 수 있습니다.
    {: .prompt-info}

[author]: 1234035817765142548/1234162109378592919/2024-04-28_220715.png?ex=662fba88&is=662e6908&hm=7ac42299797526fe27ccc334fcb9c0bfa399c9efa46176f584fd2eb10ef9c7b2&=&format=png&quality=lossless 
[contact]: 1234035817765142548/1234162109634449418/2024-04-29_001742.png?ex=662fba88&is=662e6908&hm=d6cc1b53ccfec32d255145e506c2a86a71678dadf9da0b46e00685470c231d91&=&format=png&quality=lossless
[sidebar]: 1234035817765142548/1234162109886103652/2024-04-28_221651.png?ex=662fba88&is=662e6908&hm=f8e4d5134b7dfa14bc7ea26ded7c863bf54d48b93056dfe8bf21549b60f87c95&=&format=png&quality=lossless
[profile]: 1234035817765142548/1234162110095822999/2024-04-29_001816.png?ex=662fba88&is=662e6908&hm=98a83e0fdf08dc3681542ec5d8a09d64735d50b6d4ec87c7934cdb8f59bdfc68&=&format=png&quality=lossless
[favicon]: 1234035817765142548/1234168353342623794/2024-04-29_004413.png?ex=662fc059&is=662e6ed9&hm=7dfbff93fca12d431cbefbe3af388dfc0b92ea93dc321d0be699abdc0480dfc0&=&format=png&quality=lossless
[link-hover]: 1234035817765142548/1234162110381162566/2024-04-28_014210.png?ex=662fba88&is=662e6908&hm=ebdd8ab753d8499723f9d98937097cedfaec62d5c0ca0bec504149de7c8d5028&=&format=png&quality=lossless 
[tag-hover]: 1234035817765142548/1234162110687215746/2024-04-28_014232.png?ex=662fba88&is=662e6908&hm=109f03320859b690ca79dbecaf95c3aa759976b2652b173dd2d260782e372d59&=&format=png&quality=lossless
[post-tag]: 1234035817765142548/1234162110968238181/2024-04-28_014911.png?ex=662fba88&is=662e6908&hm=c03bbd165dda23fdaa462060efea8f20e90507a10c9e013ab738bdfa31d2f9af&=&format=png&quality=lossless&width=372&height=470
[issue]: 1234035817765142548/1234162111228411954/2024-04-28_230124.png?ex=662fba88&is=662e6908&hm=8e5240e44629a0523c99edff14ece4b7f518ca26ba1c10471607f23aa06b0a01&=&format=png&quality=lossless
[label]: 1234035817765142548/1234162111463161926/2024-04-28_230340.png?ex=662fba88&is=662e6908&hm=629c46053d4f00166a6f8dddebc37550d9370be6f0538ea9435c07422b898c9c&=&format=png&quality=lossless
[label-html]: 1234035817765142548/1234162111672881302/2024-04-28_232439.png?ex=662fba89&is=662e6909&hm=056c440477a92c8de2f6dd66eba2ce6d769f6cb91770924776ab3054c4eca2f0&=&format=png&quality=lossless 
[comment]: 1234035817765142548/1234162414686048348/2024-04-28_233231.png?ex=662fbad1&is=662e6951&hm=41329b7195f864743b196b4dc6c1ae6855dad50fff5ab3f3cd58adfd01fbf359&=&format=png&quality=lossless
[comment-issue]: 1234035817765142548/1234165094204571750/2024-04-28_233314.png?ex=662fbd50&is=662e6bd0&hm=3f269f0256c8f2d1331ff7b958c12729a3e8bbc4a0ee3cc4a71c81120323cfa7&=&format=png&quality=lossless&width=807&height=471
[gct-signup]: 1234035817765142548/1234162415193821205/2024-04-28_020417.png?ex=662fbad1&is=662e6951&hm=a0f1767cbf9c0522f3ab11aa2b05c3bcb5c8a1520cda0b238284a88980c5f6ab&=&format=png&quality=lossless&width=772&height=471
[gct-settings]: 1234035817765142548/1234162415541813410/2024-04-28_020646.png?ex=662fbad1&is=662e6951&hm=03195396f6ec5c2bd1d8b95962b3d2dbe82452095e9f0c07614f86d09655298a&=&format=png&quality=lossless&width=961&height=341
[gct-dashboard]: 1234035817765142548/1234162415957184643/2024-04-28_212753.png?ex=662fbad1&is=662e6951&hm=4837bcfb3511e6b9483690fce5fa638f395914daf1b8022c3a7f9c14da1c62a5&=&format=png&quality=lossless&width=738&height=470
[pageview]: 1234035817765142548/1234162416267300895/2024-04-28_234819.png?ex=662fbad1&is=662e6951&hm=9944db3880e4ffc1e479f64e171bde7776cf0ff6d1a1a1fb0b9ed6592c47ec27&=&format=png&quality=lossless