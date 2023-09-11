---
layout: post
title: "How to use git flow"
date: 2022-04-15
categories: [Tool, Git]
tags: ["git", "git flow"]
comments: true
---


reference : [https://danielkummer.github.io/git-flow-cheatsheet/index.ko_KR.html](https://danielkummer.github.io/git-flow-cheatsheet/index.ko_KR.html)

prerequisite : 

- git
- git flow

# getting started

```jsx
git flow init
```

```bash
Which branch should be used for bringing forth production releases?
   - heads/1.2201.1
   - 1.2204.1
   - 1.2204.1-1
   - 1.2204.1-2
   - feature/securitypatch
   - master
   - nc_on_premise
   - new_architecture
Branch name for production releases: [master] master

Which branch should be used for integration of the "next release"?
   - heads/1.2201.1
   - 1.2204.1
   - 1.2204.1-1
   - 1.2204.1-2
   - feature/securitypatch
   - nc_on_premise
   - new_architecture
Branch name for "next release" development: [] feature/securitypatch

How to name your supporting branch prefixes?
Feature branches? [feature/] 
Bugfix branches? [bugfix/] 
Release branches? [release/] 
Hotfix branches? [hotfix/] 
Support branches? [support/] 
Version tag prefix? []
Hooks and filters directory? [/mnt/d/IdeaProjects/backend/argos-rpa-la-report-api/.git/hooks]
```

TroubleShooting : 

```bash
Fatal: Working tree contains unstaged changes. Aborting.
```

> git stash 

### To change branch name

```bash
$ vi .git/config
```

## Features

### start a new feature

```bash
git flow feature start MYFEATURE
```

### finish up a feature

```bash
git flow feature finish MYFEATURE
```

### publish a feature

```bash
git flow feature publish MYFEATURE
```

### Getting a published feature

```bash
git flow feature pull origin MYFEATURE
git flow feature track MYFEATURE
```

## Make a release

### start a release

```bash
git flow release start RELEASE [BASE]
git flow release publish RELEASE
```

### Finish up a release

```bash
git flow release finish RELEASE
```

## Hotfixes

### git flow hotfix start

```bash
git flow hotfix start VERSION [BASENAME]
```

### finish a hotfix

```bash
git flow hotfix finish VERSION
```

### 

### How to delete branch

```bash
git branch -d MYFEATURE
```