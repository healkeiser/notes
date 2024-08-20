---
title: Auto Changelog
icon: material/refresh
slug: auto-changelog
tags:
  - development
  - javascript
  - python
  - package
publish: true
---

_How to generate an automatic changelog, for example in a [Python package](../Python/Distribute%20Package%20to%20PyPi.md)._

## Install

Install `auto-changelog` using `npm`.

```shell
npm install -g auto-changelog
```

Create  a `package.json` file at the root of your package.

```json title=package.json
{
  "scripts": {
    "changelog": "auto-changelog -p"
  }
}
```

Let's now assume you have tagged a new release on Git, you can now run the following command to update the `CHANGELOG.md` file:

```shell
npm run changelog
```