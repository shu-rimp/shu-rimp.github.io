---
title: github.io 블로그 튜토리얼 - (1) 시작하기
author: shurimp
date: 2024-04-28 15:34:00 +0900
categories: [Blog, Notes]
tags: [setup, jekyll, github-pages]
image:
  path: posts/1-1.png
  alt: github blog
---

이번에 깃블로그를 시작하면서, 설정을 마치고 보니 한번에 모두 끝낼 수 있는 실전압축요약정리문서...같은게 있었으면 좋겠다 싶어서 작성합니다.

크게 세 파트로 나누어서 [(1) 시작하기](/posts/create-github-blog-1), [(2) 블로그 단장하기](/posts/create-github-blog-2), [(3) 포스팅](/posts/create-github-blog-3) 으로 진행합니다.

## Jekyll 테마 선택하기
깃허브 페이지는 단순히 깃허브를 통한 퍼블릭 웹 페이지를 제공하므로, 장인정신을 발휘하고 싶지 않다면 잘 만들어진 테마를 가지고 와야 합니다. 일반적으로 jekyll 기반 테마를 많이 이용하며, 그 중 <kbd>chirpy</kbd> 테마를 가장 추천합니다.

![chirpy](posts/1-1.png)
_chirpy theme_

깔끔한 디자인과 뭐 하나 빠짐없는 알찬 구성요소, 반응형까지 지원하는 테마입니다. 유저 수도 많아 커뮤니티도 활성화 되어 있습니다.

### chirpy repository fork하기
[chirpy repository](https://github.com/cotes2020/jekyll-theme-chirpy)에 접속해서 레포지토리를 포크해옵니다. 이 때 레포지토리 이름을 `<github닉네임>.github.io`로 설정합니다.

**레포지토리 공개설정은 public이어야 합니다.**

![repo](posts/1-2.png){: .left} ![repo2](posts/1-3.png){: .right}

### repository 설정
포크해온 깃헙 레포지토리의 <kbd>settings</kbd> 메뉴에 들어갑니다.

![general](posts/1-4.png)

<kbd>General</kbd> - Default branch를 `main`으로 변경합니다.

![page](posts/1-5.png)

<kbd>Pages</kbd> - Build and deployment에서 Source를 `Github Actions`로 선택합니다.

1-2분 기다리면 상단에 github.io 링크가 생성됩니다.

페이지에 접속 후 모든 기능들이 제대로 작동하는지 먼저 확인합니다.

## 로컬 환경 구성
자유롭게 편집할 수 있도록 로컬 환경을 구성합니다.

### ruby 설치
jekyll은 ruby로 개발되었으므로, ruby를 먼저 설치해야합니다. **3버전 이상**이 필요합니다.

```shell
brew install ruby
ruby -v # 버전 확인
```
윈도우 환경에서도 chocolatey로 간편하게 설치 가능합니다.(관리자 권한 필요)
> 버전 확인이 안될 시 터미널을 닫았다 열면 조회됩니다.
{: .prompt-info}

```shell
choco install ruby
ruby -v # 버전 확인
```

ruby 설치 완료 후 jekyll을 설치합니다.
```shell
gem install bundler jekyll
jekyll -v # 버전 확인
```

### nodejs 설치
nodejs **v20버전 이상**이 필요합니다. nvm 또는 npm으로 nodejs를 설치합니다.

```shell
nvm install v20.12.2
nvm use 20.12.2
node -v # 버전 확인
```

### 로컬 서버 실행
이전에 생성한 깃헙 레포지토리를 로컬로 클론해옵니다.

```shell
git clone https://github.com/shu-rimp/shu-rimp.github.io.git
```

프로젝트 폴더가 위치한 곳에서 gem 모듈을 설치합니다.
```shell
cd /workspace/shu-rimp.github.io
bundle
```

nodejs 모듈을 설치합니다. **windows에서는 명령어 수행 전 cross-env 설정이 필요합니다.**

```shell
npm install
npm run build
```

#### cross-env 설정방법
1. <kbd>cross-env</kbd> 패키지 설치
    ```shell
    npm install cross-env
    ```

2. package.json파일의 build부분 NODE_ENV 앞에 cross-env 붙인 후 저장
    ```json
    "scripts": {
        "prebuild": "npx rimraf assets/js/dist",
        "build": "cross-env NODE_ENV=production npx rollup -c --bundleConfigAsCjs",
    ``` 
    {: file='/package.json'}

모든 설치가 완료되었다면 서버를 실행합니다. `--livereload` 옵션은 소스를 수정할 때마다 자동으로 새로고침되는 옵션으로, 편집할 때 좋습니다.
```shell
jekyll serve --livereload
```
![jekyll](posts/1-6.png)

`http://127.0.0.1:4000/` 으로 접속해 블로그가 잘 보이는지 확인합니다.

![blog](posts/1-7.png)

## 깃허브에 배포하기
1. 깃허브 레포지토리 설정에서 <kbd>Pages</kbd> - Build and deployment 메뉴 GitHub Actions를 선택하면, Configure 버튼이 생깁니다. 해당 버튼 클릭 후 나타나는 창에서 <kbd>Commit changes...</kbd>를 선택합니다.

2. 로컬 레포지토리 .gitignore 파일의 assets/js/dist를 주석처리 합니다.
    ```
    # IDE configurations
    .idea

    # Misc
    # assets/js/dist # 주석
    ```
    {: file='/.gitignore'}

3. 로컬 레포를 push하기 전, 변경사항을 pull해온 후 push 합니다.
    ```shell
    git pull
    git add .
    git commit -m 'Initial Commit'
    git push
    ```

4. 깃허브 액션에서 워크플로우를 확인합니다.
![workflow](posts/1-8.png)


5. 내 블로그 주소(github.io)로 접속해 블로그가 잘 보이는지 확인합니다.
![blog](posts/1-9.png)

이제 로컬에서 작업한 변경사항을 깃허브에 push할 때마다 블로그가 업데이트됩니다.

## 커스텀 도메인 연결하기
카페24, 가비아 등 도메인 업체에서 구매한 도메인이 있다면 블로그에 연결할 수 있습니다.

먼저 구매한 도메인 사이트에서 <kbd>A레코드</kbd>와 <kbd>CNAME</kbd> 설정이 필요합니다.

### 도메인 설정
![gabia](posts/1-10.png)

사이트 내 도메인 관리 페이지에서 다음과 같이 A레코드와 CNAME을 설정합니다. A레코드는 깃허브에서 제공한 ip주소이고, CNAME의 값에는 자신의 github.io 도메인을 입력합니다. 호스트명 설정은 사이트마다 상이하므로 구매한 곳의 안내에 따라 설정합니다.(저는 **가비아**에서 구매했습니다.) CNAME 호스트 www 설정을 하지 않으면 www.shurimp.com으로 접속했을 때 페이지를 찾지 못합니다.

```text
185.199.108.153	
185.199.109.153	
185.199.110.153	
185.199.111.153
```

### 깃허브 도메인 연결

레포지토리 <kbd>Settings</kbd> - <kbd>Pages</kbd> - Custom domain에 구매한 도메인을 입력하고 <kbd>Save</kbd> 버튼을 누르면 DNS 체크가 진행되고 일정시간이 지나면 초록색 체크 표시로 바뀝니다. Enforce HTTPS에 체크하면 자동으로 인증서 발급도 해줍니다.(let's encrypt)

![dns1](posts/1-11.png)

![dns2](posts/1-12.png)

도메인으로 접속이 잘 되는지 확인합니다.

![domain](posts/1-13.png)



---

References
- [Jekyll Chirpy(v6.0.1) 테마를 활용한 Github 블로그 만들기(2023.6 기준)](https://jjikin.com/posts/Jekyll-Chirpy-%ED%85%8C%EB%A7%88%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-Github-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EB%A7%8C%EB%93%A4%EA%B8%B0(2023-6%EC%9B%94-%EA%B8%B0%EC%A4%80)/)
- [NODE-📚-cross-env-모듈-사용법](https://inpa.tistory.com/entry/NODE-%F0%9F%93%9A-cross-env-%EB%AA%A8%EB%93%88-%EC%82%AC%EC%9A%A9%EB%B2%95)
- [Managing a custom domain for your GitHub Pages site](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)









