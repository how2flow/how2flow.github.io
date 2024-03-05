---
permalink: /posts/deb-deployment/
title: "[Linux] Debian 패키지 배포하기"
excerpt: "데비안 패키지 배포하는 방법입니다. (with launchpad)"
header:
  teaser: /assets/posts/images/deb-deployment.webp
categories:
  - Linux
tags:
  - apt
  - autoconf
  - automake
  - deb
  - debhelper
  - debian
  - debuild
  - devscripts
  - dpkg
  - launchpad
  - packaging
  - ppa
  - ubuntu
toc: true
---

[데비안 패키지를 만들었으면](/documents/linux/deb-packaging/), PPA를 만들고 배포해야 합니다.<br>
launchpad를 사용해서 간단하게 PPA를 만들고 배포하는 과정을 정리합니다.<br>
Git repo 기준으로 작성되었습니다.<br>

## 패키지 배포하기

패키지를 먼저 배포 하기 위해서 PPA를 가지고 있어야 합니다.<br>
대략적인 패키지 배포 순서입니다.
```
1. launchpad 계정 만들기
2. PPA 만들기
3. 프로젝트 만들기
4. 코드 import
5. 레시피 만들기
```

### launchpad 계정 만들기

PPA는 launchpad 계정 만들고 난 후 생성합니다.<br>

[계정 생성/만들기](https://login.launchpad.net/+login)

계정을 만들고 다음 페이지에 접속합니다.<br>
```
https://launchpad.net/~{user_name}
```

제 계정 페이지 입니다.<br>

<p align="center">
  <img src="/assets/posts/images/launchpad-overview.png" alt="launchpad-overview" width="640" height="480"><br>
  <span style="{{ site.img }}">[picture 1] Launchpad Overview</span>
</p>
<br>

좌측 하단에 있는 <span style="{{ site.code }}">Create a new PPA</span> 을 클릭하여 PPA를 만듭니다.<br>


### project 만들기

[프로젝트](https://launchpad.net/projects/+new)를 만들어서 패키지를 관리해야 합니다.<br>

<p align="center">
  <img src="/assets/posts/images/launchpad-project.png" alt="launchpad-project" width="640" height="480"><br>
  <span style="{{ site.img }}">[picture 2] Launchpad Project</span>
</p>
<br>

위 사진은 <span style="{{ site.code }}">this-is-testing</span> 이름으로 프로젝트를 추가 하는 경우 입니다.<br>
작성이 끝났으면 <span style="{{ site.code }}">continue</span> 버튼으로 다음과정을 진행합니다.<br>

Check for duplication projecsts 페이지가 나오면 기존에 존재하는 프로젝트 이름과 비슷해서 나오는 페이지 입니다.<br>
이름을 수정하거나, <span style="{{ site.code }}">No, this is a new project</span> 버튼으로 다음 과정으로 넘어가면 됩니다.<br>

<p align="center">
  <img src="/assets/posts/images/launchpad-project-detail.png" alt="launchpad-project-detail" width="640" height="480"><br>
  <span style="{{ site.img }}">[picture 3] Launchpad Project Detail</span>
</p>
<br>

프로젝트 관련해서 추가로 작성하는 페이지 입니다.<br>
<span style="{{ site.code }}">Description</span>, <span style="{{ site.code }}">Homepage URL</span>, <span style ="{{ site.code }}">Licenses</span> 등을 추가로 작성하고<br>
하단에 <span style="{{ site.code }}">Complete Registration</span> 버튼으로 프로젝트를 생성합니다.<br>

### 코드 import 하기

[launchpad 코드 import 페이지](https://code.launchpad.net/+code-imports/+new)로 접속합니다.<br>

<p align="center">
  <img src="/assets/posts/images/launchpad-import.png" alt="launchpad-import" width="640" height="480"><br>
  <span style="{{ site.img }}">[picture 4] Launchpad Code Imports</span>
</p>
<br>

<span style="{{ site.code }}">Project</span> 에는 전 과정에서 만든 프로젝트 이름을 작성합니다.<br>
위의 이미지는 Git 기준으로 선택되어 있습니다.<br>
코드 관리를 <span style="{{ site.code }}">Bazzar</span> 를 사용할지 <span style="{{ site.code }}">Git</span>을 사용할 지 선택하고 코드의 URL을 작성해 줍니다.<br>

<span style="{{ site.code }}">Request Import</span> 로 import를 완료합니다.<br>

### 레시피 만들기

레시피를 만들 패키지 페이지로 접속합니다.<br>
패키지 페이지는 레포지토리 리스트에서 접속 할 수 있습니다.<br>
다음 사진은 Git 레포지토리 리스트 접속 하기 전 모습입니다.<br>

<p align="center">
  <img src="/assets/posts/images/launchpad-git-repo.png" alt="launchpad-git-repo" width="640" height="480"><br>
  <span style="{{ site.img }}">[picture 5] Launchpad Git 레포지토리 리스트</span>
</p>
<br>

레포지토리 리스트를 통해 패키지 페이지로 접속합니다.<br>
저는 기존에 사용하고 있는 패키지로 접속을 해서 보이는 화면이 조금 다를 수 있습니다.<br>

<p align="center">
  <img src="/assets/posts/images/launchpad-package-page.png" alt="launchpad-package-page" width="640" height="480"><br>
  <span style="{{ site.img }}">[picture 6] Launchpad 패키지 페이지</span>
</p>
<br>

패키지 사이트에 접속하면, <span style="{{ site.code }}">Create packaging recipe</span>으로 접속합니다.<br>

접속 후, 나오는 페이지 하단에 다음과 같이 나옵니다.<br>

<p align="center">
  <img src="/assets/posts/images/launchpad-recipe.png" alt="launchpad-recipe" width="640" height="480"><br>
  <span style="{{ site.img }}">[picture 7] Launchpad Create packaging recipe</span>
</p>
<br>

페이지 하단에서 <span style="{{ site.code }}">distribution</span>과 <span style="{{ site.code }}">Recipe text</span>를 작성합니다.<br>

<span style="{{ site.code }}">Recipe text</span>는 다음과 같이 작성합니다.
```
# git-build-recipe format 0.4 deb-version {debupstream}-0~{revtime}
lp:~{user_name}/{project_name}/+git/{git_repo_name} master
```

작성 예시입니다.
```
# git-build-recipe format 0.4 deb-version {debupstream}-0~{revtime}
lp:~how2flow/how2flow-apps/+git/kernel-scripts master
```
