---
title: "Dev Containers로 구성하는 안정적인 개발 환경"
author: shurimp
date: 2025-03-20 20:57:00 +0900
categories: ["Infra", "Docker"]
tags: [dev-containers,docker,jekyll]
# 이미지 삽입 시 아래 줄 주석 제거
image: 
  path: posts/4-0.png
  alt: dev containers
---

## 주요 키워드
- Dev Containers란?
- devcontainer.json
- 사용 방법 #1: 컨테이너에 직접 Clone
- 사용 방법 #2: 로컬에 Clone 후 컨테이너로 Reopen
- 두 방식의 차이와 주의사항

---

## 💥윈도우에서는 안되는데요...
처음 이 블로그를 구성할 때는 MacOS 환경에서 구성했습니다. 별 다른 수고 없이 그냥 튜토리얼만 잘 따라해도 깔끔한 블로그가 만들어지는 것을 보고 돌아볼 것 없이 기존에 사용하던 티스토리에서 갈아탔습니다. 그리고 몇달 후, PC를 Windows로 바꾸면서 문제가 시작됐습니다. 분명 for Windows용 가이드대로 했는데도 수많은 충돌과 빌드 에러, 의존성 문제로 필요없는 시간을 너무 많이 낭비해 버렸습니다.

![4-1][4-1]
_무한 오류 지옥_

하나를 해결하면 또 하나가 문제되고, 기존에 잘 되던게 안되니까 짜증은 나는데 내가 블로그 때문에 이걸 붙잡고 있어야 하나라는 생각과 함께 그냥 쓰던거나 쓸 걸 이라는 후회도 됐습니다.

그러던 중, 사용중인 Jekyll 테마 문서에서 <kbd>Dev Containers</kbd>를 활용하는 방법을 우연히 접하게 되었고... 한 번에 완벽히 실행되는 블로그를 보고 감탄하게 되었습니다. "더 알아봐야겠다!"하고 좀 더 자세히 파보게 된 점을 이 포스트에 정리합니다.

## Dev Containers란?
Microsoft에서 만든 **오픈 규격**으로, 개발에 특화된 라이브러리, 툴, 설정들을 컨테이너 이미지로 관리하고 누구나 동일한 환경에서 작업할 수 있도록 합니다. 여러 툴을 지원하지만 VS Code의 확장 프로그램으로 많이 사용합니다.

도커를 기반으로 하며, `.devcontainer/devcontainer.json` 파일로 환경을 정의합니다. Dev Containers를 사용하면 얻는 이점으로는 다음과 같은 것들이 있습니다.
- 개발자의 운영체제와 무관하게 동일한 개발 환경 보장
- 의존성 충돌 최소화
- 로컬 디렉토리에 간섭하지 않음
- 템플릿을 통해 개발자간 환경 설정을 공유하고 재사용할 수 있음
- 개발환경과 운영환경의 분리

## devcontainer.json
Dev Containers는 `.devcontainer/devcontainer.json` 설정 파일을 기준으로 작동합니다. 이 설정 파일에는 사용할 Docker 이미지(또는 Dockerfile, docker-compose.yml), 컨테이너 시작시 실행할 명령어, 포트 포워딩, 마운트 설정, VS Code 익스텐션 설치 등 컨테이너를 어떻게 구성할 것인지 지정할 수 있는 필드들이 있습니다.

사용할 수 있는 모든 필드는 문서에서 확인할 수 있습니다.[https://containers.dev/implementors/json_reference/](https://containers.dev/implementors/json_reference/)

아래는 주로 사용하는 필드 몇가지 입니다.
```json
{
  "name": "My Dev Container",  // 컨테이너 이름
  "image": "mcr.microsoft.com/devcontainers/javascript-node:20", // 사용할 Docker 이미지
  "onCreateCommand": "git config --global --add safe.directory ${containerWorkspaceFolder}", // 컨테이너 생성 시 실행
  "postCreateCommand": "npm install", // 컨테이너 생성 후 실행 (예: 의존성 설치)
  "forwardPorts": [3000], // 컨테이너에서 호스트로 노출할 포트
  "customizations": {
    "vscode": {
      "extensions": [ // 자동 설치할 VS Code 확장
        "esbenp.prettier-vscode",
        "dbaeumer.vscode-eslint"
      ],
      "settings": {
        "editor.formatOnSave": true // 저장 시 자동 포맷
      }
    }
  }

  // "runArgs": ["--cap-add=SYS_PTRACE"], // docker run에 넘기는 인자
  // "remoteUser": "vscode",              // 기본 사용자
  // "mounts": ["source=path,target=/workspace,type=bind"] // 볼륨 마운트
}
```
{: file='.devcontainer/devcontainer.json'}



## 사용 방법 #1: 컨테이너에 직접 Clone
Dev Container에 원격 레포지토리를 직접 복제해서 작업합니다.
- 컨테이너가 실행될 때 볼륨을 함께 생성하고 해당 볼륨에 레포지토리를 복제하고 작업합니다.
- 컨테이너 볼륨을 사용하기 때문에 로컬 디렉토리에 영향을 주지 않습니다.
- **단, 볼륨 삭제 시 모든 변경사항이 사라집니다.(저장을 위해서는 git push 필수)**

![4-3][4-3]

(VS Code) `F1` -> `Dev Containers: Clone Repository in Container Volume...`              
컨테이너에 제 github.io 레포지토리를 복제했습니다.

![4-4][4-4]

`Docker Desktop`에서 컨테이너와 볼륨이 생성된 것을 확인할 수 있습니다.

![4-5][4-5]

여기서 `test-container.md` 파일을 생성해도, 내 로컬 디렉토리에는 생성되지 않습니다. 이 데이터는 컨테이너 내 볼륨에 위치해있기 때문입니다.

![4-6][4-6]

컨테이너를 종료하고, 생성된 볼륨을 삭제했습니다.

![4-7][4-7]

레포지토리를 다시 복제해오면, 새로운 컨테이너와 볼륨이 생성됐기 때문에 `test-container.md`파일은 존재하지 않습니다. 내 로컬 디렉토리에 있는 데이터들은 아무 영향을 받지 않습니다.


## 사용 방법 #2: 로컬에 Clone 후 컨테이너로 Reopen

먼저 로컬에 레포지토리를 복제한 후, `Reopen in Container`로 열 수 있습니다.

- 작업은 컨테이너 내에서 하고, 볼륨은 로컬 디렉토리와 연결됩니다.
- 따라서 변경 사항이 로컬 디렉토리에 반영됩니다.
- **컨테이너를 삭제해도 내 PC에서 변경사항이 그대로 유지됩니다.**

![4-8][4-8]

이번에는 로컬 폴더를 VS Code로 열고 컨테이너로 Reopen 합니다. 이 알림창은 `.devcontainer/devcontainer.json` 파일이 있는 경우 자동으로 나타납니다.

![4-10][4-10]

1번 방식과는 다르게, 컨테이너만 생성하고 볼륨은 생성되지 않는 것을 확인할 수 있습니다.

![4-9][4-9]

컨테이너 내에서 `test-local.md` 파일을 생성하면, 로컬 디렉토리에서도 파일을 확인할 수 있습니다.


## 두 방식의 차이와 주의사항

| 항목          | 컨테이너에 Clone (볼륨 사용)           | 로컬에 Clone 후 Reopen (바인딩)        |
| ------------- | -------------------------------------- | -------------------------------------- |
| 디렉토리 연결 | 로컬과 완전 분리됨 (바인딩 없음)       | 로컬 디렉토리와 연결 (바인딩됨)        |
| 변경사항 반영 | Git push 전까지는 로컬에 없음          | 변경 즉시 로컬 디스크에 저장됨         |
| 안전성        | 컨테이너/볼륨 삭제 시 데이터 손실 위험 | 컨테이너 삭제해도 로컬 데이터는 보존됨 |
| 속도          | 빠름 (도커 볼륨 기반)                  | 느릴 수 있음 (로컬 디스크 접근)        |
| 권장 용도     | 실험, PR 리뷰, 테스트용                | 블로그 작성, 실 프로젝트 개발 등       |


로컬 디렉토리에 계속 저장할 필요가 없는 테스트 용도로는 속도가 빠른 1번 방식을 사용해서 테스트 후 깔끔하게 삭제하고, 계속 저장이 필요하면서 운영환경의 충돌을 방지하기 위한 용도로 사용할 경우 2번 방식을 선택하는 것이 좋아 보입니다. 저는 블로그 용도로 사용하기 때문에 2번을 선택했습니다.

필요에 따라서 1, 2번 방식을 모두 사용할 수 있습니다. 예를 들어 블로그에 테마를 변경하거나 플러그인을 추가하고 싶을 때, 브랜치를 나누는 것도 좋은 방법이지만 1번 방법을 사용해서 내가 원하는 환경을 그대로 가져와 테스트 후 삭제하거나, 마음에 들면 PR을 날려 반영할 수 있습니다. 

VS Code의 익스텐션이지만, Intellij Ultimate 버전에서도 Dev Container를 지원합니다. Spring을 사용하는 백엔드 프로젝트에서도 유용하게 사용할 수 있을 것 같습니다.

## 마무리

블로그를 직접 운영하니 생각보다 사소한 오류를 자주 접하게 됩니다. 때로는 중요하지 않은 것에 시간을 낭비하고 있다는 생각도 합니다. 하지만 문제를 해결함과 동시에 매력적인 도구를 찾아내고, 더 깊게 알아보는 과정에서 보람을 느낍니다. 앞으로 어플리케이션 개발에도 사용해보려고 합니다. 



[4-0]: posts/4-0.png
[4-1]: posts/4-1.png
[4-3]: posts/4-3.png
[4-4]: posts/4-4.png
[4-5]: posts/4-5.png
[4-6]: posts/4-6.png
[4-7]: posts/4-7.png
[4-8]: posts/4-8.png
[4-9]: posts/4-9.png
[4-10]: posts/4-10.png
