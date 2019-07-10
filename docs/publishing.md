---
id: publishing
title: Publishing
sidebar_label: Publishing
---

发布原生模块是原生模块的关键部分。原生节点模块的用户，与其计算机的体系结构兼容。 Node社区为此设计了两种解决方案

### 1. 上传和下载原生编译的模块

> 此方法要求您拥有Travis CI帐户。如果你一个都没有的话， [please create one](https://travis-ci.com).

[Example](https://github.com/amilajack/disk-utility)

库作者可以将原生模块编译到所有多个目标（windows，macOS，linux），然后上传这些目标。然后，该模块的用户将下载这些目标。

要实现此解决方案，请将以下.travis.yml添加到项目的根目录：

```yaml
os:
  - osx
  - linux

language: node_js

node_js:
  - node
  - 10
  - 9
  - 8

cache: cargo

before_install:
  # Install Rust and Cargo
  - curl https://sh.rustup.rs -sSf > /tmp/rustup.sh
  - sh /tmp/rustup.sh -y
  - export PATH="$HOME/.cargo/bin:$PATH"
  - source "$HOME/.cargo/env"
  # Install NPM packages
  - node -v
  - npm -v
  - npm install

script:
  - npm test
  # Publish when using '[publish binary]' keywords
  - COMMIT_MESSAGE=$(git log --format=%B --no-merges -n 1 | tr -d '\n')
  - if [[ ${COMMIT_MESSAGE} =~ "[publish binary]" ]]; then yarn upload-binary || exit 0; fi;
```

然后将以下行添加到`package.json`，告诉NPM只发布`lib`目录和`native / index.node`文件。 确保更改`repository`对象中的`type`和`url`属性：

```json
  "repository": {
    "type": "git",
    "url": "git+https://github.com/your-username-here/your-project-here.git"
  },
  "files": [
    "native/index.node",
    "lib"
  ],
```

然后安装[`node-pre-gyp`]（https://github.com/mapbox/node-pre-gyp）和[`node-pre-gyp-github`]（https://github.com/bchr02/节点预GYP-github上）。 请注意，`node-pre-gyp`的主分支不能与Neon一起使用，所以你必须使用fork的分支来增加对Neon的支持：

```bash
# NPM
npm install node-pre-gyp@amilajack/node-pre-gyp#neon-compat
npm install node-pre-gyp-github
# Yarn
yarn add node-pre-gyp@amilajack/node-pre-gyp#neon-compat
yarn add node-pre-gyp-github
```

Then make the following changes to the `scripts` section of your `package.json`:
```diff
- "install": "neon build --release",
+ "install": "node-pre-gyp install --fallback-to-build=false || neon build --release",
+ "package": "node-pre-gyp package",
+ "upload-binary": "node-pre-gyp package && node-pre-gyp-github publish",
```

`node-pre-gyp install --fallback-to-build = false`将尝试下载二进制文件而不是回退到使用`node-pre-gyp`的项目构建。 以下部分，`|| 如果前一个脚本抛出错误，则使用Neon构建生产关系。

Finally, add the following to the root of your `package.json`:

```json
  "binary": {
    "module_name": "index",
    "host": "https://github.com/your-username-here/your-repo-here/releases/download/",
    "remote_path": "{version}",
    "package_name": "{node_abi}-{platform}-{arch}.tar.gz",
    "module_path": "./native",
    "pkg_path": "."
  },
```

这通过告诉我们`native / index.node`在哪里以及在哪里上传二进制文件来配置`node-pre-gyp`。 确保用实际值替换`your-username-here`和`your-repo-here`。

注意：请勿将“{version}”替换为实际版本。

GitHub需要获得代表您发布版本的权限，因此您需要创建个人访问令牌：

1. Go to [Developer Settings](https://github.com/settings/developers)
2. Click `Personal access tokens`
3. Click `Generate new token`
4. Select `"public_repo"` and `"repo_deployment"`
5. Generate Token
6. Copy the key that's generated and set NODE_PRE_GYP_GITHUB_TOKEN environment variable to it. Within your command prompt:

Then add the key to the Travis CI settings of your repo.

1. Open your project on Travis CI
2. Click `More options` dropdown
3. Click `Settings`
4. Go to `Environment Variables` and add `NODE_PRE_GYP_GITHUB_TOKEN` as `Name` and your generated token as the `Value` of your ENV variables

For an **example of a Neon project with publishing capabilities**, see [amilajack/disk-utility](https://github.com/amilajack/disk-utility)
For more details on [`node-pre-gyp-github`'s `README`](https://github.com/bchr02/node-pre-gyp-github) for more details on publishing options

现在，您只能在特定提交上发布二进制文件。 为此，您可以借用Travis CI关于commit关键字的想法，并使用[publish binary]为提交消息添加特殊处理：

```bash
git commit -a -m "[publish binary]"
```

Or, if you don't have any changes to make simply run:

```bash
git commit --allow-empty -m "[publish binary]"
```

请注意，这些将运行`upload-binary`脚本，该脚本将打包并上载当前版本的包的二进制文件。 发布Neon模块的典型工作流程包括以下内容：

```bash
git commit -a -m "[publish binary]"
npm publish
```

### 2. Shipping compiling native modules

另一个解决方案是使用npm模块传送所有已编译的本机代码目标。 换句话说，这意味着使用windows，macOS和linux二进制文件发布包。 这种解决方案被认为比上传和下载模块的方法更安全，因为它们可以防止模块的用户受到虫洞攻击。 [wormhole attacks](https://www.kb.cert.org/vuls/id/319816/).

This feature is still a work in progress as the [`neon build --target` feature](https://github.com/neon-bindings/rfcs/issues/16) is a work in progress.
