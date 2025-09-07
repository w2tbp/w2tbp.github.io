---
title: "使用Obsidian、Hugo、Github Pages来写博客"
date: 2024-10-24T13:10:30+08:00
lastmod: 2025-09-07T18:20:31+08:00
---

现在一般用 obsidian 来写东西，但复制到博客里面部署就有点麻烦，所幸就写了一个插件来转换。然后顺便给 hugo 换个主题啥的。于是就在部署到 Github Pages 的过程中，踩了半天坑。

## Hugo 初始化
跟着文档一步一步来就好：[Quick start](https://gohugo.io/getting-started/quick-start/)

我没有使用包管理器，直接在 github 上下载的 release。要注意的是，hugo 是分版本的，下载那个 extended 版本。由于我是 windows，就下载 windows-amd64。
下载链接：[Releases · gohugoio/hugo](https://github.com/gohugoio/hugo/releases)

为了能在命令行使用，可以将 hugo 的安装路径写到系统环境变量 path 中。

然后就可以用命令行来创建项目了，官方文档提示，最好使用 PowerShell，不要使用 CMD 或 Windows PowerShell：[Installing PowerShell on Windows](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.5)
既然它这么说，肯定有它的道理，下下也无妨。我看了一眼，这玩意应该算 Windows PowerShell 的升级版，有命令自动补全之类的功能（方向键→触发补全）

运行以下命令，就可以得到一个用  [Ananke](https://github.com/theNewDynamic/gohugo-theme-ananke) 主题创建的 hugo 空项目。
```powershell
hugo new site quickstart
cd quickstart
git init
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
echo "theme = 'ananke'" >> hugo.toml
hugo server
```

由于 anake 感觉不够好看，于是换成 [Blowfish](https://themes.gohugo.io/themes/blowfish/)
安装主题后，发现它和我现有的 hugo 主题不兼容，于是下载了历史版本的 hugo

```
git submodule add -b main https://github.com/nunocoracao/blowfish.git themes/blowfish
```

需要将 `themes/_defalt ` 中的内容复制到根目录来，然后再删除根目录的 `hugo.toml`，这样配置才能正确生效。

blowfish 配置：[入门指南 · Blowfish](https://blowfish.page/zh-cn/docs/getting-started/)

## Obsidian 插件
也是从其他人那里得到的灵感，这个插件的主要作用就是找到 Obsidian 库里带 blog 标签的文章，然后转换成 hugo 的文件头格式，再丢到 hugo 的文章目录下。

插件地址：[w2tbp/obsidian-hugo-convert](https://github.com/w2tbp/obsidian-hugo-convert)

## Github Pages 部署
在部署前，需要新建一个以 `用户名.github.io` 为名字的项目，并将代码推到这个仓库中。

从[托管和部署 · Blowfish](https://blowfish.page/zh-cn/docs/hosting-deployment/) 这里找到一个 workflow，在项目的 `.github/workflows/` 目录下新建一个 `gh-pages.yml` 文件。

```yaml
# .github/workflows/gh-pages.yml

name: GitHub Pages

on:
  push:
    branches:
      - master

jobs:
  build-deploy:
    runs-on: ubuntu-24.04
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: true
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: "latest"

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: ${{ github.ref == 'refs/heads/master' }}
        with:
          github_token: ${{ secrets.ACCESS_TOKEN }}
          publish_branch: gh-pages
          publish_dir: ./public

```

其中，我更改了分支的名字（main -> master），并且更改了 `github_token: ${{ secrets.ACCESS_TOKEN }}`，这也是我接下来踩到的坑。

### acces token
这个 token 是 github 为了安全而做的东东：[在工作流中使用 GITHUB_TOKEN 进行身份验证 - GitHub 文档](https://docs.github.com/zh/actions/tutorials/authenticate-with-github_token) 在 github action 中，需要这个 token 来进行身份验证，操作代码库。

我们在 settings -> Developer Settings -> Personal access tokens -> Fine-grained personal access tokens 中可以新建。

点击生成按钮后，就进入了填表环节，有一些需要注意的。

1. 选择代码库访问权限，是访问所有代码库，还是选定的代码库，这里我只选择了自己需要的。

![](使用Obsidian、Hugo、Github%20Pages来写博客-20250907.png)

2. 选择权限，这个 token 将权限分的很细，这也是踩坑的地方。我们的 action 中需要推送代码带 gh-pages 分支，所以需要代码库的读写权限。

![](使用Obsidian、Hugo、Github%20Pages来写博客-20250907%201.png)

这样，一个 token 就生成好了，将它复制，然后到 `用户名.github.io` 项目中选择 settings -> secrets and variables -> actions -> New repository secret 

新建一个名为 ACCESS_TOKEN 的 secret，并把 token 复制过去。搞定。

不知道会不会存在由于 gh-pages 不存在而导致 action 失败的情况，如果出现了这种情况，可以使用以下方法：

```bash
# 进入 public/ 目录并初始化 Git 
cd public 
git init 
git remote add origin https://github.com/你的用户名/你的用户名.github.io.git 
git checkout -b gh-pages 

# 提交构建结果并推送到 gh-pages 分支 
git add . 
git commit -m "手动部署 Hugo 页面" 
git push -f origin gh-pages
```

### 配置 github pages
在 settings -> pages 中按如下配置：

![](使用Obsidian、Hugo、Github%20Pages来写博客-20250907%202.png)

