---
id: electron-apps
title: Electron Apps
sidebar_label: Electron Apps
---

[Examples](https://github.com/neon-bindings/examples/tree/master/electron-app)

Neon可以很好地为Electron应用程序添加本机功能。本指南将向您介绍一个向Electron应用程序添加Neon依赖项的简单示例。要查看整个示例，您可以查看完整的演示。[full demo](https://github.com/neon-bindings/examples/tree/master/guides/electron-apps/simple-app).

在大多数情况下，在Electron中使用Neon包就像将它添加到package.json依赖项一样简单。但是，Electron确实为构建原生模块带来了自己的要求。

**我们正在努力为 [electron-rebuild]添加Neon支持(https://github.com/electron/electron-rebuild)**, 因此您可以像其他任何一样将Neon依赖项放入您的应用中。 目前，有一个名为electron-build-env的工具可用于构建Electron应用程序的任何Neon依赖项。

与此同时，我们可以在我们的package.json中使用几行配置的Electron应用程序中使用Neon模块。

## Setup Electron

```bash
# Clone the Quick Start repository
git clone https://github.com/electron/electron-quick-start

# Go into the repository
cd electron-quick-start
```

## 增加一个 Neon 依赖

首先让我们添加一个简单的Neon模块，neon-hello，它已经在npm中发布：

```bash
npm install neon-hello
```

## 添加构建依赖项

接下来，我们需要使用neon-cli和electron-build-env软件包来构建neon-hello。由于它们只需要构建，我们可以将它们添加为开发依赖项：

```bash
npm install electron-build-env neon-cli --save-dev
```

## Creating a Build Script

最后，我们将添加一个脚本来构建Neon依赖项：

```jsonc
"scripts": {
    // ...
    "build": "electron-build-env neon build neon-hello --release"
    // ...
}
```

此步骤使用electron-build-env工具正确配置环境以构建Electron。

## That's It!

然后我们可以构建neon-hello模块的生产版本

```shell
npm run build
```

我们应该有一个工作的 Electron app. 您可以通过构建和运行来试用完整的工作演示： [full working demo](https://github.com/neon-bindings/examples/tree/master/electron-app)

```shell
npm start
```
