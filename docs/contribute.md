# 加入我们！怎么参与到这个项目中？

!!! tip "基础知识"
    本项目源代码托管于 GitHub，使用 Markdown 语法编写，使用 [MkDocs Material](https://www.mkdocs.org/) 生成静态网页。如果你不熟悉这些工具，可以参考 [GitHub 官方文档](https://docs.github.com/cn) 和 [MkDocs Material 官方文档](https://squidfunk.github.io/mkdocs-material/)。

## 加入 GitHub 私有仓库

本仓库由[朱老板](https://github.com/Bwoah-Kimi/)管理，如果你想加入这个项目，可以联系朱老板，他会将你加入到 GitHub 私有仓库中。

## 本地 Git 账户配置

在本地配置 Git 账户，以便在本地提交代码。

```
git config --global user.name  "Your Name"
git config --global user.email "Your E-mail"
```

!!! warning "用户名和邮箱"
    请务必提前设置好，否则脚本抓取贡献者时会出现错误。

## 本地部署

你需要下载如下几个 python 包，用于本地部署网页。

```
pip install mkdocs-material
pip install mkdocs-git-revision-date-localized-plugin
pip install mkdocs-git-authors-plugin
pip install mkdocs-git-committers-plugin-2
pip install mkdocs-encryptcontent-plugin
```

将本仓库克隆到本地，然后在仓库根目录下执行 `mkdocs serve`，然后在浏览器中输入 `http://127.0.0.1:8000` 即可预览网页。

## 添加新内容

如果你想添加一个单独的页面，可以在 `docs` 目录下新建一个 `.md` 文件，然后在 `mkdocs.yml`、`index.md` 文件中添加该文件的路径。

!!! warning "添加图片"
    如果你想添加图片，请将图片放在 `docs/assets/images` 目录中，并在 `.md` 文件里按当前文件位置使用相对路径引用（例如 `assets/images/xxx.png` 或 `../assets/images/xxx.png`）。

## 提交代码

在本地修改完代码后，可以使用如下命令提交代码。

```
git add .
git commit -m "Your commit message"
git push
```
