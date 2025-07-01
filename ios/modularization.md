---
layout: default
title: Modularization of an iOS app
categories: swift
---


### Creating shared product

1. add submodule to your product `git submodule add git@github.com:artur-gurgul-pro/sharepack.git Sharepack`
1. Extract shared code to multi-module package. See `Sharepack`
2. Add local package

TBD


Initialisation of a library

```bash
mkdir MyDeps && cd MyDeps
swift package init --type library
```

```bash
swift package generate-xcodeproj
```

```bash
xcodebuild -resolvePackageDependencies
```
