## 如何使用

1、安装`commitlint`和`commitlint-config`

```bash
# Install and configure if needed
npm install --save-dev @commitlint/{cli,config-conventional}
echo "module.exports = { extends: ['@commitlint/config-conventional'] };" > commitlint.config.js
```

2、安装哈士奇（husky）

```bash
# Install Husky v6
npm install husky --save-dev
# or
yarn add husky --dev

# Active hooks
npx husky install
# or
yarn husky install

# Add hook
npx husky add .husky/commit-msg 'npx --no-install commitlint --edit $1'
# or
yarn husky add .husky/commit-msg 'yarn commitlint --edit $1'
```

3、测试

```bash
git commit -m "foo: this will fail"
husky > commit-msg (node v10.1.0)
No staged files match any of provided globs.
⧗   input: foo: this will fail
✖   type must be one of [build, chore, ci, docs, feat, fix, perf, refactor, revert, style, test] [type-enum]

✖   found 1 problems, 0 warnings
ⓘ   Get help: https://github.com/conventional-changelog/commitlint/#what-is-commitlint

husky > commit-msg hook failed (add --no-verify to bypass)
```

# commit规范

[TOC]

## commit信息结构

### 1、概述

  commit 信息的结构如下所示：

```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

  日常工作中提交 commit ，建议尽量细化每次的提交内容，例如修复一个 bug、完成一个新功能后，就提交一次commit，使提交记录更加清晰、细致，方便后续项目代码的维护、查看。

### 2、type

  type用于直观表明本次 commit 的作用。普遍参考的 [Angular.js 的规范](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines)，主要设置有如下几种类型：

- feat：新功能（feature）
- fix：修复bug
- docs：文档类更改
- style：不影响代码运行、构建等逻辑的格式上的优化（代码空格数量、缺失分号等）
- refactor：既不是新功能，也不是修bug的代码更改
- perf：性能优化
- chore：构建过程、辅助工具的更改

### 3、scope[可选]

  scope说明本次 commit 的影响范围，一般指代码位置、所属模块等。建议在 `type` 为 `fix` 时，此处填写修复的 bug 编号，方便之后的查询。

### 4、subject

  subject是 commit 信息的描述，相比 body 更为简短。

### 5、body[可选]

  body是 commit 信息的详细描述。如果 subject 足够描述清楚，则 body 可以省略不写。

### 6、footer[可选]

  footer包含重大更改、关联的 issues 等内容。

## 示例

  以下是 Angular.js 给出的[例子](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.8sw072iehlhg)：

```
feat($browser): onUrlChange event (popstate/hashchange/polling)

Added new event to $browser:
- forward popstate event if available
- forward hashchange event if popstate not available
- do polling when neither popstate nor hashchange available

Breaks $browser.onHashChange, which was removed (use onUrlChange instead)
fix($compile): couple of unit tests for IE9

Older IEs serialize html uppercased, but IE9 does not...
Would be better to expect case insensitive, unfortunately jasmine does
not allow to user regexps for throw expectations.

Closes #392
Breaks foo.bar api, foo.baz should be used instead
```

  去 Github 上查看 Angular 的提交记录可以看到，最基础的 commit 有 type、scope、subject 三个部分即可。