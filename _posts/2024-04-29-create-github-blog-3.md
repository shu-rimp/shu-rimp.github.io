---
title: github.io 블로그 튜토리얼 - (3) 포스팅
author: shurimp
date: 2024-04-29 15:34:00 +0900
categories: [Blog, Notes]
tags: [setup, jekyll, github-pages]
image:
  path: posts/1-1.png
  alt: github blog
---

> **이전 포스팅 바로가기** <br>
> [(1) 시작하기](/posts/create-github-blog-1) <br>
> [(2) 블로그 단장하기](/posts/create-github-blog-2) <br>
> (3) 포스팅 <br>

---

이제 가장 중요한 포스팅에 대해 작성합니다.

## 포스트 작성하기
포스트는 <kbd>/_posts</kbd> 디렉토리 내에 마크다운 파일으로 작성합니다.

![3-1][3-1]

포스트 이름은 마음대로 정하면 됩니다. 저는 관리가 용이하도록 `YYYY-MM-DD-title-0.md` 형식으로 지정했습니다. 

### 프론트매터 작성
포스트 최상단에서 다양한 정보를 입력할 수 있습니다.

```yaml
---
title: github.io 블로그 튜토리얼 - (3) 포스팅
author: shurimp
date: 2024-04-29 15:34:00 +0900
categories: [Blog, Notes]
tags: [setup, jekyll, github-pages]
image:
  path: posts/1-1.png
  alt: github blog
---
```
<kbd>title</kbd>: 포스트 제목. 여기에 작성한 제목으로 웹에 보여집니다.               
<kbd>author</kbd>: 작성자. 이전에 <kbd>authors.yml</kbd>에서 작성자 설정을 했다면 이름을 클릭했을 때 원하는 링크로 연결되는 기능을 지원합니다.                        
<kbd>date</kbd>: 포스트 작성 날짜. iso 8601 형식으로, UTC이기 때문에 timezone을 지정해야합니다.                
<kbd>categories</kbd>: 카테고리. 새 카테고리를 입력하면 별도의 설정없이 새로운 카테고리를 생성합니다. depth는 2개까지만 지원합니다.               
<kbd>tags</kbd>: 태그를 달 수 있습니다(태그 검색도 지원). 태그는 갯수 제한 없이 여러개 지정 가능합니다.                 
<kbd>image</kbd>: 대표 이미지를 지정할 수 있습니다. path에 이미지 경로를 넣고 alt에 대체 텍스트를 지정합니다.               


### 글 작성
글은 기본적으로 마크다운 문법을 사용합니다.
스타일은 테마의 고유 스타일이 적용됩니다.
제가 쓰는 chirpy 테마의 경우 튜토리얼로 제공하고 있습니다.
[https://chirpy.cotes.page/posts/text-and-typography/](https://chirpy.cotes.page/posts/text-and-typography/)

기본적인 문법 몇 가지만 소개합니다.
```md
<!-- 제목, 갯수가 많아질수록 소제목 -->
# h1
## h2
### h3
#### h4 

<!-- 코드블록, 언어 지정 가능 -->
    ```java
    // Enter Code here
    ```
{: file='/_config.yml'} <!-- 코드블록의 파일명 지정 가능 -->

<!-- 키보드(단어 강조) -->
<kbd>words</kbd>

<!-- 링크 첨부 -->
[표시텍스트](링크)

<!-- 이미지 첨부 -->
![대체텍스트](경로)

<!-- 인용 -->
> 텍스트

<!-- 프롬프트 블록 -->
> 텍스트
{: .prompt-info} <!-- [info|tip|warning|danger] -->
```



### 이미지 관리
이미지 경로 입력을 간편화하기 위해 해당 테마에서는 prefix 기능을 제공합니다.

```yaml
# The CDN endpoint for media resources.
# Notice that once it is assigned, the CDN url
# will be added to all media resources (site avatar, posts' images, audio and video files) paths starting with '/'
#
# e.g. 'https://cdn.com'
cdn: "/assets/img"
```
{: file='/_config.yml'}

<kbd>config.yml</kbd> 파일 내 cdn 옵션에 중복되는 경로를 지정하면 매번 길게 지정하지 않아도 이미지 첨부가 가능합니다. 이름에서 알 수 있듯이, 이미지가 많아 외부 CDN을 사용하는 경우에도 해당 주소를 입력하면 모든 이미지 로드시 앞에 경로를 붙여서 탐색합니다.
<p/>
만약 특정 포스트에서만 필요한 경로가 있을 때는, 해당 포스트의 프론트매터에 경로를 지정할 수 있습니다.

```yaml
media_subpath: /path/to/media/
```
둘 다 지정 시, `{cdn}/{media_subpath}/{path}` 순서로 경로가 작성됩니다.

#### 이미지 첨부 팁
마크다운 문법을 약간 응용한 팁입니다. 이미지 첨부 시, 포스트 최하단에 이미지 경로를 모아놓고 필요한 부분에서 별칭으로 사용하는 방법을 추천합니다.             
해당 포스트에 사용된 이미지를 관리하기 매우 쉽고, 나중에 모종의 이유로(이미지 용량이 커져서 저장소를 옮겨야 한다거나, 경로가 바뀌거나) 수정해야 할 때 한번에 편집하기 용이합니다.      

```md
![author](author) <!--필요한 곳에 사용-->

...

[author]: posts/2-1.png
[contact]: posts/2-2.png
[sidebar]: posts/2-3.png
```

[3-1]: posts/3-1.png
