---
title: 从零搭建Hexo博客并部署到GitHub
tags:
  - Hexo
  - 博客
categories:
  - Hexo相关
cover: 'https://img.liuranblog.top/file/1789225567040_hexo_dj.png'
abbrlink: 6b218ca2
date: 2026-09-12 14:52:51
---
你是否曾有过这样的想法 —— 想拥有一个属于自己的技术博客，却苦于没有服务器、不会维护数据库？如果你有 WordPress 博客的搭建经验，可能会被服务器费用、域名备案、安全维护等问题困扰。今天我要介绍的方法，可以让你零成本拥有一个完全属于你的个人博客，而且只需要几分钟就能上线。

# 为什么选择 GitHub Pages + Hexo？
在开始之前，让我们先搞清楚为什么这个方案值得尝试。
**GitHub Pages** 是 GitHub 提供的免费静态网站托管服务，每个账户可以托管一个名为 *用户名.github.io* 的仓库，访问地址就是 *https://用户名.github.io* 更重要的是，它是完全免费的，而且自带 CDN 加速。
**Hexo** 是一个快速、简洁且高效的博客框架，它的特点是：
 -  静态生成 —— 所有页面都是 HTML 文件，无需服务器
 -  Markdown 支持 —— 用最熟悉的语法写博客
 - 丰富主题 —— 社区提供了大量精美主题
 - 插件生态 —— 支持 RSS、搜索、评论等各种功能
这两者的结合，意味着你只需要写 Markdown，就能得到一个高性能、高颜值的技术博客。

# 环境准备
在开始之前，你需要准备以下环境：
## 安装 Node.js
Hexo 基于 Node.js 运行，所以第一步是安装 Node.js。访问 [Node.js 官网](https://nodejs.org/zh-cn)，下载 LTS（长期支持）版本，直接运行安装程序即可。
```cmd
# 验证安装
node --version
# 输出类似: v20.x.x
```
## 安装 Git
Git 用于将博客代码推送到 GitHub。Windows 用户建议安装 [Git for Windows](https://git-scm.com/)，安装时选择 “Use Git from Git Bash only”
```git
# 验证安装
git --version
# 输出类似: git version 2.x.x
```
## 创建 GitHub 仓库
登录 [GitHub](https://github.com/)，点击右上角的 “+” → “New repository”，创建一个名为 **你的用户名.github.io** 的公开仓库。例如我的用户名是 **Liuran**，就创建 **Liuran.github.io**。
要勾选 “Initialize this repository with a README”
> 💡 小贴士：仓库名必须是 用户名.github.io 的格式，GitHub 才会自动将其识别为 Pages 站点。
## 简单配置
进入到你要安装的博客目录，我的是D:\blog，然后右键选择 **Open Git Bash here**
### 设置用户名和邮箱
```cmd
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的GitHub注册邮箱@example.com"
```
### 创建 SSH 密匙
```cmd
ssh-keygen -t rsa -C "GitHub 注册邮箱"
```
输入之后一直回车就行
### 添加密匙
进入 [C:\Users\用户名\.ssh] 目录（要勾选显示“**隐藏的项目**”），用记事本打开公钥 **id_rsa.pub** 文件并复制里面的内容。
登陆 GitHub ，进入 Settings 页面，选择左边栏的 **SSH and GPG keys**，点击 New SSH key。
Title 随便取个名字，粘贴复制的 id_rsa.pub 内容到 Key 中，点击 Add SSH key 完成添加。
### 验证连接
打开 Git Bash，输入
```cmd
ssh -T git@github.com  出现 “Are you sure……”，输入 yes 回车确认。
```
显示 **“Hi xxx! You've successfully……”** 即连接成功。

# 本地安装 Hexo 博客程序
新建一个文件夹用来存放 Hexo 的程序文件，如 Hexo-Blog。打开该文件夹，右键选择 **Open Git Bash Here**。
## 安装 Hexo
使用 npm 一键安装 Hexo 博客程序
```cmd
npm install -g hexo-cli
```
Mac 用户需要管理员权限（sudo），运行这条命令
```mac
sudo npm install -g hexo-cli
```
安装时间有点久（真的很慢！），界面也没任何反应，耐心等待，如果失败可以使用科学上网工具
## Hexo 初始化和本地预览
```cmd
hexo init      # 初始化
npm install    # 安装组件
```
完成后输入下面命令，**启动本地服务器进行预览**
```cmd
hexo g && hexo s
```
访问 http://localhost:4000，出现 Hexo 默认页面，本地博客安装成功！
> 💡 本地预览： 此时输入 hexo s 或者 hexo server | npx hexo server，并在浏览器打开localhost:4000，即可看到你的博客页面。在终端按 Ctrl + C 可停止预览。

![hexo初始界面](https://img.didadi.xyz/file/1784624990270_localhost.png)

> 💡如果出现页面加载不出来，可能是端口被占用了。Ctrl+C 关闭服务器，运行 hexo server -p 5000 更改端口号后重试。

# 部署 Hexo 到 GitHub Pages
本地博客测试成功后，就是上传到 GitHub 进行部署，使其能够在网络上访问。
首先安装 **hexo-deployer-git**
```cmd
npm install hexo-deployer-git --save
```
然后修改 **_config.yml** 文件末尾的 **Deployment** 部分，修改成如下
```yml
deploy:
  type: git
  repository: git@github.com:用户名/用户名.github.io.git
  branch: main
```
完成之后运行三件套就行了
```cmd
hexo clean && hexo generate && hexo deploy
也可以简化成
hexo cl && hexo g && hexo d
```
完成！这时访问我们的 GitHub 域名 https://用户名.github.io 就可以看到 Hexo 网站了。
## 便捷指令
为了方便操作可以在博客目录找到**package.json**文件打开在"server": "hexo server",的下面添加
```json
"a": "hexo clean && hexo generate && hexo server",
"b": "hexo clean && hexo generate && hexo deploy",
"c": "hexo clean && hexo generate",
```
之后在终端就可以使用
```cmd
npm run a    #启动本地浏览
npm run b    #三连git到Github Page
npm run c    #快速重构
```
# 绑定域名（可选）
博客搭建完成使用的是 GitHub 的子域名（用户名.github.io），我们可以为 Hexo 博客绑定自己的域名替换 GitHub 域名，更加个性化和专业，也利于 SEO。
## 添加解析
注册好你的专属域名后，添加DNS记录
记录类型选择**CNAME**，记录值为 你的用户名.github.io
## 绑定域名到 Hexo 博客
进入本地博客文件夹的 source 目录，打开记事本，里面输入自己的域名，如 `www.example.com`，保存名称为 “CNAME”，格式为 “所有文件”（无 .txt 后缀）
清除缓存等文件并重新发布网站，现在就可以使用自己的域名访问 Hexo 博客了。
## 开启 HTTPS
配置自己的域名后，需要我们手动开启 HTTPS。打开博客所在 GitHub 仓库，Settings -> 下拉找到 GitHub Pages -> 勾选 Enforce HTTPS。
HTTPS 证书部署成功需要一定时间，等大概几分钟再访问域名，就可以看到域名前面的小绿锁了，HTTPS 配置完成！

# 开始使用
## 发布文章
进入博客所在目录，右键打开 Open Git Bash Here，创建博文
```cmd
hexo new "My New Post"
```
然后 source 文件夹中会出现一个 My New Post.md 文件，就可以使用 Markdown 编辑器在该文件中撰写文章了。
写完后运行下面代码将文章渲染并部署到 GitHub Pages 上完成发布。以后每次发布文章都是这两条命令。
```cmd
hexo g && hexo d
```
也可以不使用命令自己创建 .md 文件，只需在文件开头手动加入如下格式 Front-matter 即可，写完后运行命令发布。
```md
---
title: Hello World # 标题
date: 2019/3/26 hh:mm:ss # 时间
categories: # 分类
- Diary
tags: # 标签
- 1
- 2
---

摘要
<!--more-->
正文
```
## 网站设置
包括网站名称、描述、作者、链接样式等，全部在网站目录下的 _config.yml 文件中，参考官方文档按需要编辑。

> 💡 冒号后要加一个空格！

## 常用命令
```cmd
hexo new "name"       # 新建文章
hexo new page "name"  # 新建页面
hexo g                # 生成页面
hexo d                # 部署
hexo g -d             # 生成页面并部署
hexo s                # 本地预览
hexo clean            # 清除缓存和已生成的静态文件
hexo help             # 帮助
```
# 结语
Hexo 是一种纯静态的博客，我们必须要在本地完成文章的编辑再部署到 GitHub 上，依赖于本地环境。不能像 WordPress 或 Typecho 那样的动态博客一样能直接在浏览器中完成撰文和发布。
可以说是一种比较极客的写博客方式，但是优势也是明显的——免费稳定省心，比较适合爱折腾研究的用户，或者没有在线发文需求的朋友。