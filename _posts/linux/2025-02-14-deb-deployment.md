---
categories:
- linux
excerpt: How to deploy Debian packages using Launchpad.
header:
  teaser: /assets/images/posts/thumbnails/note.jpg
layout: single
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
title: '[Linux] Deploying Debian Packages'
toc: true
redirect_from:
- /documents/linux/deb-deployment/
- /legacy/deb-deployment/
- /documents/linux/deb-deployment
- /legacy/deb-deployment
---
Once you have [created a Debian package](/linux/deb-packaging/), you need to create a PPA and deploy it.<br>
This guide summarizes the process of creating a PPA and deploying it using Launchpad.<br>
Written based on a Git repository.<br>

## Deploying Packages

To deploy a package, you must first have a PPA.<br>
The general deployment sequence is as follows:
```
1. Create a Launchpad account
2. Create a PPA
3. Create a project
4. Import code
5. Create a recipe
```

### Creating a Launchpad Account

A PPA is created after setting up a Launchpad account.<br>

[Create Account](https://login.launchpad.net/+login)

After creating an account, visit the following page:<br>
```
https://launchpad.net/~{user_name}
```

This is my account page.<br>

<p align="center">
  <img src="/assets/images/posts/contents/linux/launchpad-overview.png" alt="launchpad-overview" width="640" height="480"><br>
  <span style="{{ site.img }}">[picture 1] Launchpad Overview</span>
</p>
<br>

Click <span style="{{ site.code }}">Create a new PPA</span> at the bottom left to create your PPA.<br>


### Creating a Project

You need to create a [project](https://launchpad.net/projects/+new) to manage your packages.<br>

<p align="center">
  <img src="/assets/images/posts/contents/linux/launchpad-project.png" alt="launchpad-project" width="640" height="480"><br>
  <span style="{{ site.img }}">[picture 2] Launchpad Project</span>
</p>
<br>

The image above shows the process of adding a project named <span style="{{ site.code }}">this-is-testing</span>.<br>
Once completed, click the <span style="{{ site.code }}">Continue</span> button to proceed to the next step.<br>

The "Check for duplication projects" page appears if the name is similar to existing projects.<br>
You can either modify the name or click <span style="{{ site.code }}">No, this is a new project</span> to continue.<br>

<p align="center">
  <img src="/assets/images/posts/contents/linux/launchpad-project-detail.png" alt="launchpad-project-detail" width="640" height="480"><br>
  <span style="{{ site.img }}">[picture 3] Launchpad Project Detail</span>
</p>
<br>

This page is for additional project details.<br>
Fill in the <span style="{{ site.code }}">Description</span>, <span style="{{ site.code }}">Homepage URL</span>, <span style ="{{ site.code }}">Licenses</span>, etc.,<br>
and click the <span style="{{ site.code }}">Complete Registration</span> button at the bottom to create the project.<br>

### Importing Code

Access the [Launchpad code import page](https://code.launchpad.net/+code-imports/+new).<br>

<p align="center">
  <img src="/assets/images/posts/contents/linux/launchpad-import.png" alt="launchpad-import" width="640" height="480"><br>
  <span style="{{ site.img }}">[picture 4] Launchpad Code Imports</span>
</p>
<br>

Enter the project name created in the previous step into the <span style="{{ site.code }}">Project</span> field.<br>
The image above shows Git as the selected option.<br>
Choose whether to use <span style="{{ site.code }}">Bazaar</span> or <span style="{{ site.code }}">Git</span> for code management and provide the repository URL.<br>

Complete the import by clicking <span style="{{ site.code }}">Request Import</span>.<br>

### Creating a Recipe

Access the package page to create a recipe.<br>
The package page can be accessed from the repository list.<br>
The following image shows the Git repository list before access.<br>

<p align="center">
  <img src="/assets/images/posts/contents/linux/launchpad-git-repo.png" alt="launchpad-git-repo" width="640" height="480"><br>
  <span style="{{ site.img }}">[picture 5] Launchpad Git Repository List</span>
</p>
<br>

Access the package page through the repository list.<br>
I accessed a package I'm already using, so your screen might look slightly different.<br>

<p align="center">
  <img src="/assets/images/posts/contents/linux/launchpad-package-page.png" alt="launchpad-package-page" width="640" height="480"><br>
  <span style="{{ site.img }}">[picture 6] Launchpad Package Page</span>
</p>
<br>

On the package site, click <span style="{{ site.code }}">Create packaging recipe</span>.<br>

After accessing, the following will appear at the bottom of the page.<br>

<p align="center">
  <img src="/assets/images/posts/contents/linux/launchpad-recipe.png" alt="launchpad-recipe" width="640" height="480"><br>
  <span style="{{ site.img }}">[picture 7] Launchpad Create packaging recipe</span>
</p>
<br>

Enter the <span style="{{ site.code }}">distribution</span> and <span style="{{ site.code }}">Recipe text</span> at the bottom of the page.<br>

Write the <span style="{{ site.code }}">Recipe text</span> as follows:
```
# git-build-recipe format 0.4 deb-version {debupstream}-0~{revtime}
lp:~{user_name}/{project_name}/+git/{git_repo_name} master
```

Example:
```
# git-build-recipe format 0.4 deb-version {debupstream}-0~{revtime}
lp:~how2flow/how2flow-apps/+git/kernel-scripts master
```
