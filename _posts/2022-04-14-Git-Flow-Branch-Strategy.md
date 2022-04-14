---
layout: post
title: "How to Debug React on VS Code"
date: 2022-04-01
excerpt: "How to Debug React on VS Code"
tags: ["Debugging", "React", "VSCode"]
comments: true
---


Currently I am working alone on RPA API project. At first I didn’t realize for the need of git strategy for Gitlab (we are using Github) but as time passes, even though I am the only one who pushes to and pulls from main branches, I sometimes have conflicts and get confused.  

Even before I started to work here, there was only one person who manages the project. So no strategy was made. 

**Better late than never!** 

While I was googling about git strategy, Google showed me the way : GIt FLow 

# Git branch model

reference : 

[A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/#supporting-branches, "A successful Git branching model")

## The main branches

- `master`
- `develop`

When the source code in the `develop` branch reachs a stable point and is ready to be released, all of the changes should be mereged back into `master` somehow and then tagged with a release number.

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0401/Untitled.png">

## Supporting branches

- Features branches
- Release branches
- Hotfix branches

### Feature branches

May branch off from : develop

Must merge back into : develop

Branch naming convention: anything except `master`, `develop`, `release-*`, or `hotfix-*`

Feature branches are used to develop new features for the upcoming or a distant future release. The essence of a feature branch is that it exists as long as the feature is in development, but will eventually be merged back into `develop` or discarded. 

Feature branches typically exist in developer repos only, not in `origin`.

**Incorporating a finished feature on develop**

Finished features may be merged into the develop branch to definitely add them to the upcoming release : 

```jsx
$ git checkout develop
$ git merge --no-ff myfeature
$ git branch -d myfeature
Delete branch myfeature
$ git push origin develop
```

The `--no-ff` flag cuases the merge to always create a new commit object, event if the merge could be performed with a fast-forward. This avoids losing information about the historical existence of a feature branch and groups together all commits that together added the feature. Compare: 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0401/Untitled%201.png">

### Release branches

May branch off from : `develop`

Must merge back into: `develop` and `master`

Branch naming conveiton: `release-*`

Release branches support preparation of a new production release. The key moment to branch off a new release branch from develop is when develop (almost) reflects the desired state of the new release. It is exactly at the start of a release branch that the upcoming release gets assigned a version number —not any earlier. 

### Hotfix branches

May branch off from: `master`

Must merge back into: `develop` and `master`

Branch naming convention: `hotfix-*`

Hotfix branches are very much like release branches in that they are also meant to prepare for a new production release, albeit unplanned. When a critical bug in a production version must be resolved immediately, a hotfix branch may be branched off from the corresponding tag on the master branch that marks the production version. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0401/Untitled%202.png">

The one exception to the rule here is that, **when a release branch currently exists, the hotfix changes need to be merged into that release branch, instead of `develop`.**

Comprehensive flowchart: 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0401/Untitled%203.png">