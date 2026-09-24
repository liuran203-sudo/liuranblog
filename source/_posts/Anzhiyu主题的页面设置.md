---
title: Anzhiyu主题的页面设置
tags:
  - Hexo
  - 安知鱼主题
  - 博客
categories:
  - Hexo
cover: 'https://img.liuranblog.top/file/1790269367957_post_3.png'
abbrlink: '75815004'
date: 2026-09-22 00:17:50
---
## 1.前言
Anzhiyu 内置大量特色独立页面，不需要自己手写页面布局，只需要简单生成页面文件、配置数据源、开启导航菜单，就可以实现丰富的站点功能。
本篇一次性配置：**友人帐 (友链)、留言板、我的装备、关于页面、追番页、朋友圈、相册集、闲言碎语（即刻短文）**。
> ⚠️重要提醒：所有 yml 配置，**禁止使用 Tab 制表符，必须用空格缩进，冒号后面保留空格**，否则页面报错空白。
## 2.进阶配置
### 2.1.友人帐页面
友人帐是 Anzhiyu 主题的友链专属页面，支持多种卡片样式、分组管理、站点预览图，同时自带友链提交表单，是技术博客很常用的页面。
#### 2.1.1.配置
**步骤 1：生成友链页面**
在 Hexo 博客**根目录**打开终端（Git Bash / PowerShell 都可以），执行命令：
```bash
hexo new page link
```
执行成功后，在`source`目录会出现`link`文件夹，路径 `source/link/index.md`，打开这个文件。
**步骤 2：修改 index.md 的 Front-matter**
把文件内容替换为下面代码，你可以自行修改标题、日期、页面简介：
```markdown
---
title: link
date: 2020-12-01 22:19:45
type: "link"
---
```
> ✅重点：`type: "link"` 这一行不能删除、不能写错，少引号或者拼写错误，页面不会渲染友链卡片。
**步骤 3：编写友链数据源 link.yml**
在 `source/_data/` 目录新建文件 `link.yml`，所有友链分组、博主信息全部写在这里。
```yml
- class_name: 框架
  flink_style: flexcard
  hundredSuffix: ""
  link_list:
    - name: Hexo
      link: https://hexo.io/zh-tw/
      avatar: https://d33wubrfki0l68.cloudfront.net/6657ba50e702d84afb32fe846bed54fba1a77add/827ae/logo.svg
      descr: 快速、简单且强大的网站框架
    - name: anzhiyu主题
      link: https://hexo.anheyu.com/
      avatar: https://npm.elemecdn.com/anzhiyu-blog-static@1.0.4/img/avatar.jpg
      descr: 生活明朗，万物可爱
      siteshot: https://npm.elemecdn.com/anzhiyu-theme-static@1.1.6/img/hexo.anheyu.com.jpg

- class_name: 推荐博客
  flink_style: telescopic
  hundredSuffix: ""
  link_list:
    - name: 安知鱼
      link: https://hexo.anheyu.com/
      avatar: https://npm.elemecdn.com/anzhiyu-blog-static@1.0.4/img/avatar.jpg
      descr: 生活明朗，万物可爱
      siteshot: https://npm.elemecdn.com/anzhiyu-theme-static@1.1.6/img/hexo.anheyu.com.jpg
      color: vip
      tag: 技术

- class_name: 小伙伴
  class_desc: 那些人，那些事
  flink_style: anzhiyu
  hundredSuffix: ""
  link_list:
    - name: 安知鱼
      link: https://hexo.anheyu.com/
      avatar: https://npm.elemecdn.com/anzhiyu-blog-static@1.0.4/img/avatar.jpg
      descr: 生活明朗，万物可爱
      recommend: true
```
**字段详细说明**

| 参数 | 是否必填 | 说明 |
|-----|-----|-----|
| class_name | 必填 | 友链分类名 |
| class_desc | 可选 | 友链分类描述 |
| flink_style | 必填 | 卡片渲染样式，可选 `flexcard` / `anzhiyu` / `telescopic` |
| hundredSuffix | 可选 | 头像图片后缀，用于图床统一裁剪，一般留空 |
| link_list | 必填 | 当前分组下所有友链的数组列表 |
| link_list.name | 必填 | 博主昵称 |
| link_list.link | 必填 | 博客主页完整 URL |
| link_list.avatar | 必填 | 头像图片直链，优先使用稳定图床，避免失效 |
| link_list.descr | 必填 | 博客简短介绍文案 |
| link_list.siteshot | 可选 | flexcard、telescopic 模式生效，站点预览截图链接 |
| link_list.recommend	 | 可选 | 设置`true`，卡片右上角出现「荐」标识 |
| link_list.tag | 可选 | 卡片左上角自定义标签文字 |
| link_list.color | 可选 | 标签背景色，支持十六进制色号，快捷值 `vip`、`speed` |

**步骤 4：导航菜单开启友人帐入口**
打开主题配置文件 `_config.anzhiyu.yml`，找到`menu`配置块，添加友人帐条目，注意缩进：
```yml
menu:
  友链:
    友人帐: /link/ || icon-link
```
成果如图所示
![alt text](https://img02.anheyu.com/adminuploads/1/2023/04/09/6432641611b97.png!blogimg)
#### 2.1.2.与数百博主共同进步
这个功能会在友链页面顶部渲染大量博主头像墙，适合友链数量 30 条以上的站点，友链太少会大面积空白。同样在`_config.anzhiyu.yml`添加：
```yml
# 友情链接顶部相关配置
linkPageTop:
  enable: true
  title: 与数百名博主无限进步
  # 添加博主友链的评论自定义格式
  addFriendPlaceholder: "昵称（请勿包含博客等字样）：\n网站地址（要求博客地址，请勿提交个人主页）：\n头像图片url（请提供尽可能清晰的图片，我会上传到我自己的图床）：\n描述：\n站点截图（可选）：\n"
```
### 2.2.关于页面
个人介绍主页，模块丰富：头像、个人简介、技能标签、时间线生涯、兴趣爱好等。
**步骤 1：生成 about 页面**
```bash
hexo new page about
```
编辑 `source/about/index.md`
```markdown
---
title: 关于
date: 2021-03-30 15:57:51
aside: false
top_img: false
background: "#f8f9fe"
comments: false
type: "about"
---
```
> `type: "about"` 是页面渲染识别标识。
**步骤 2：编写数据源 about.yml**
在`source/_data/about.yml`编写全部个人信息，下面是完整示例，你按需修改、删减模块：
```yml
- class_name: 关于页
  subtitle: 生活明朗，万物可爱✨
  avatarImg: https://npm.elemecdn.com/anzhiyu-blog-static@1.0.0/img/avatar.webp
  avatarSkills:
    left:
      - 🤖️ 数码科技爱好者
      - 🔍 分享与热心帮助
      - 🏠 智能家居小能手
      - 🔨 设计开发一条龙
    right:
      - 专修交互与设计 🤝
      - 脚踏实地行动派 🏃
      - 团队小组发动机 🧱
      - 壮汉人狠话不多 💢
  name: 陈志伟
  description: 是一名 前端工程师、学生、独立开发者、博主
  aboutsiteTips:
    tips: 追求
    title1: 源于
    title2: 热爱而去 感受
    word:
      - 学习
      - 生活
      - 程序
      - 体验
  helloAbout: Hello there!
  skillsTips:
    tips: 技能
    title: 开启创造力
  careers:
    tips: 生涯
    title: 无限进步
    list:
      - desc: EDU,软件工程专业
        color: "#357ef5"
      - desc: EDU,软件工程专业
        color: "#357ef5"
      - desc: EDU,软件工程专业
        color: "#357ef5"
    img: https://bu.dusays.com/2023/04/21/644287166329b.png
  statistic:
    link: /archives
    text: 文章隧道
    cover: https://bu.dusays.com/2023/05/01/644f4b037b930.jpg
  map:
    title: 我现在住在
    StrengthenTitle: 中国，长沙市
    background: https://bu.dusays.com/2023/07/05/64a4c61cb20ef.jpg
    backgroundDark: https://bu.dusays.com/2023/07/05/64a4c63495ac5.jpg
  selfInfo:
    selfInfoTips1: 生于
    selfInfoContentYear: 2002
    selfInfoTips2: 湖南信息学院
    selfInfoContent2: 软件工程
    selfInfoTips3: 现在职业
    selfInfoContent3: 大三学生👨‍🎓
  personalities:
    author_name: 执政官
    personality_type: ESFJ-A
    photo_url: https://bu.dusays.com/2023/07/05/64a4c63495ac5.jpg
    personality_img: https://npm.elemecdn.com/anzhiyu-blog@2.0.8/img/svg/ESFJ-A.svg
    name_url: https://www.16personalities.com/ch/esfj-%E4%BA%BA%E6%A0%BC
  maxim:
    maxim_tips: 座右铭
    maxim_top: 生活明朗，
    maxim_bottom: 万物可爱。
  buff:
    buff_tips: 特长
    buff_top: 脑回路新奇的 酸菜鱼
    buff_bottom: 二次元指数 MAX
  game:
    game_tips: 爱好游戏
    game_title: 原神
    game_uid: "UID: 125766904"
    game_bg: https://bu.dusays.com/2023/04/22/64433bf26e25d.webp
  comic:
    comic_tips: 爱好番剧
    comic_title: 追番
    comic_list:
      - name: 约定的梦幻岛
        href: https://www.bilibili.com/bangumi/media/md5267750/?spm_id_from=666.25.b_6d656469615f6d6f64756c65.1
        cover: https://bu.dusays.com/2023/05/27/647166c44b414.webp
      - name: 咒术回战
        href: https://www.bilibili.com/bangumi/media/md28229899/?spm_id_from=666.25.b_6d656469615f6d6f64756c65.1
        cover: https://bu.dusays.com/2023/05/24/646db4398832e.webp
      - name: 紫罗兰永恒花园
        href: https://www.bilibili.com/bangumi/media/md8892/?spm_id_from=666.25.b_6d656469615f6d6f64756c65.1
        cover: https://bu.dusays.com/2023/05/24/646db43983d99.webp
      - name: 鬼灭之刃
        href: https://www.bilibili.com/bangumi/media/md22718131/?spm_id_from=666.25.b_6d656469615f6d6f64756c65.1
        cover: https://bu.dusays.com/2023/05/24/646db439856a0.webp
      - name: JOJO的奇妙冒险 黄金之风
        href: https://www.bilibili.com/bangumi/media/md135652/?spm_id_from=666.25.b_6d656469615f6d6f64756c65.1
        cover: https://bu.dusays.com/2023/05/30/64760e38d651a.webp
  like:
    like_tips: 关注偏好
    like_title: 数码科技
    like_bg: https://bu.dusays.com/2022/12/06/638f5f05ce1f7.jpg
    like_bottom: 手机、电脑软硬件
  music:
    music_tips: 音乐偏好
    music_title: 许嵩、民谣、华语流行
    music_bg: https://p2.music.126.net/Mrg1i7DwcwjWBvQPIMt_Mg==/79164837213438.jpg
    music_link: /music
  reward_list:
    - name: 海阔蓝
      amount: 8.8
      datatime: 2023-03-28
    - name: LK66
      amount: 66.6
      datatime: 2023-03-24
    - name: 张时貳
      amount: 6.6
      datatime: 2023-01-22
    - name: ZeroAf
      amount: 9.9
      datatime: 2022-12-14
    - name: LuckyWangXi
      amount: 6.6
      datatime: 2022-12-14
    - name: 刀中日月长
      amount: 10
      datatime: 2022-11-16
    - name: 鹿啵包
      amount: 10
      datatime: 2022-11-08
    - name: 疾速k
      amount: 50
      datatime: 2022-09-20
    - name: 伴舟先生大霖子
      amount: 4.03
      datatime: 2022-10-27
      suffix: 贝壳
    - name: Magica_0x0
      amount: 3.36
      datatime: 2022-10-07
      suffix: 贝壳
    - name: 名字就是要短像这样
      amount: 3.36
      datatime: 2022-08-25
      suffix: 贝壳
    - name: Leviathan520
      amount: 1.34
      datatime: 2022-08-23
      suffix: 贝壳
    - name: 托马斯
      amount: 10
      datatime: 2022-08-19
    - name: 哇是猫猫欸
      amount: 1.34
      datatime: 2022-08-19
      suffix: 贝壳
```
**步骤 3：导航菜单**
```yml
menu:
  关于:
    关于本人: /about/ || icon-zhifeiji

```
样式如图所示：
![alt text](https://img02.anheyu.com/adminuploads/1/2023/04/09/6432643720ef6.png!blogimg)

### 2.3.我的装备页面
前往你的 Hexo 博客的根目录 在 Hexo 博客根目录 [blog] 下打开终端，输入
```bash
hexo new page equipment
```
你会找到 source/equipment/index.md 这个文件
修改这个文件： 记得添加 type: "equipment"
```markdown
title: 我的装备
date: 2023-06-10 21:33:24
type: equipment
aside: false
top_img: false
```
添加数据，新建文件source\_data\equipment.yml,没有_data文件夹的话也请自己新建。以下是默认格式示例，打开source\_data\equipment.yml，输入
```yml
- class_name: 好物
  description: 实物装备推荐
  tip: 跟 安知鱼 一起享受科技带来的乐趣
  top_background: https://bu.dusays.com/2023/07/05/64a4c38842b7a.webp
  good_things:
    - title: 生产力
      description: 提升自己生产效率的硬件设备
      equipment_list:
        - name: MacBook Pro 2021 16 英寸
          specification: M1 Max 64G / 1TB
          description: 屏幕显示效果好、色彩准确、对比度强、性能强劲、续航优秀。可以用来开发和设计。
          image: https://bu.dusays.com/2023/07/05/64a4c3b191e2e.png
          link: /posts/571d.html
        - name: iPad 2020
          specification: 深空灰 / 128G
          description: 事事玩得转，买前生产力，买后爱奇艺。
          image: https://bu.dusays.com/2023/07/05/64a4c3b191e2e.png
          link: https://www.apple.com.cn/ipad-10.2/
        - name: iPhone 12 mini
          specification: 绿色 / 128G
          description: 超瓷晶面板，玻璃背板搭配铝金属边框，曲线优美的圆角设计，mini大小正好一只手就抓住，深得我心，唯一缺点大概就是续航不够。
          image: https://bu.dusays.com/2023/07/05/64a4c3ded6319.webp
          link: https://www.apple.com.cn/iphone-12/specs/
        - name: AirPods（第三代）
          specification: 标准版
          description: 第三代对比第二代提升很大，和我一样不喜欢入耳式耳机的可以入，空间音频等功能确实新颖，第一次使用有被惊艳到。
          image: https://bu.dusays.com/2023/07/05/64a4c3ded6319.webp
          link: https://www.apple.com.cn/airpods-3rd-generation/
    - title: 出行
      description: 用来出行的实物及设备
      equipment_list:
        - name: Apple Watch Series 8
          specification: 黑色
          description: 始终为我的健康放哨，深夜弹出站立提醒，不过确实有效的提高了我的运动频率，配合apple全家桶还是非常棒的产品，缺点依然是续航。
          image: https://bu.dusays.com/2023/07/05/64a4c40ab698a.webp
          link: https://www.apple.com.cn/apple-watch-series-8/
        - name: NATIONAL GEOGRAPHIC双肩包
          specification: 黑色
          description: 国家地理黑色大包，正好装下16寸 Macbook Pro，并且背起来很舒适，底部自带防雨罩也好用，各种奇怪的小口袋深得我心。
          image: https://bu.dusays.com/2023/07/05/64a4c40ab698a.webp
          link: https://item.jd.com/100011269828.html
        - name: NATIONAL GEOGRAPHIC学生书包🎒
          specification: 红白色
          description: 国家地理黑色大包，冰冰🧊同款，颜值在线且实用。
          image: https://bu.dusays.com/2023/07/05/64a4c40ab698a.webp
          link: https://item.jd.com/100005889786.html
```

主题配置文件中开启menu中关于和我的装备的注释，导航栏我的装备，注意缩进！！！
```yml
  关于:
    我的装备: /equipment/ || anzhiyu-icon-dice-d20

```

### 2.4.留言板页面
在博客根目录执行
```bash
npm install hexo-butterfly-envelope --save
```
在站点配置文件_config.yml中添加以下内容配置，更多配置请查看信笺样式留言板

```yml
#envelope_comment
#seehttps://akilar.top/posts/e2d3c450/
envelope_comment:
  enable: true #控制开关
  custom_pic:
    cover: https://npm.elemecdn.com/hexo-butterfly-envelope/lib/violet.jpg #信笺头部图片
    line: https://npm.elemecdn.com/hexo-butterfly-envelope/lib/line.png #信笺底部图片
    beforeimg: https://npm.elemecdn.com/hexo-butterfly-envelope/lib/before.png # 信封前半部分
    afterimg: https://npm.elemecdn.com/hexo-butterfly-envelope/lib/after.png # 信封后半部分
  message: #信笺正文，多行文本，写法如下
    - 有什么想问的？
    - 有什么想说的？
    - 有什么想吐槽的？
    - 哪怕是有什么想吃的，都可以告诉我哦~
  bottom: 自动书记人偶竭诚为您服务！ #仅支持单行文本
  height: #1024px，信封划出的高度
  path: #【可选】comments 的路径名称。默认为 comments，生成的页面为 comments/index.html
  front_matter: #【可选】comments页面的 front_matter 配置
    title: 留言板
    comments: true
    top_img: false
    type: envelope
```
### 2.5.首页即刻说说页面
在 Hexo 博客根目录 [blog]下打开终端，输入
```bash
hexo new page essay
```
你会找到 source/essay/index.md 这个文件
修改这个文件： 记得添加 type: "essay"
```markdown
---
title: 即刻短文
date: 2020-07-22 22:06:17
comments: true
aside: false
top_img: false
type: essay
---
```
新建source/_data/essay.yml，输入以下内容，具体字段不做解释，可以依葫芦画瓢。
```yml
- title: 即刻短文
  subTitle: 咸鱼的日常生活。
  tips: 随时随地，分享生活
  buttonText: 关于我
  buttonLink: /about/
  limit: 30
  home_essay: true
  top_background: https://img02.anheyu.com/adminuploads/1/2022/08/21/630249e2df20f.jpg
  essay_list:
    - content: 安知鱼主题指南
      date: 2023/09/09
      video:
        - https://player.bilibili.com/player.html?aid=226886152&bvid=BV1Ch41137tR&cid=1081639816&p=1&autoplay=0
    - content: 支持了Accesskey快捷键，可以直接按下shift + ?组合键以查看快捷键选项。
      date: 2023/07/01
      video:
        - https://cdn.jsdelivr.net/npm/anzhiyu-blog-static@1.0.0/video/%E9%A3%8E%E8%BD%A6%E6%A0%B7%E5%BC%8F%E6%95%88%E6%9E%9C%E9%A2%84%E8%A7%88.mp4
      image:
        - https://img02.anheyu.com/adminuploads/1/2023/07/01/64a033cb2c21e.webp!blogimg
      address: 长沙
      from: 安知鱼
      link: /posts/e140.html
    - content: 音乐支持了参数设置自定义歌单
      date: 2023/01/02
      link: https://hexo.anheyu.com/music/?id=7269231710&server=tencent
    - content: 关于页的打赏仿了b站的充电功能，使用svg绘图➕一些动画参数移动，应该不会被b站警告吧😜，另外文章也支持了顶部随机b站同款春秋冬banner。
      date: 2022/12/18
    - content: React中不能直接修改state的一个重要原因是在性能优化时的prueComponment会进行浅层比较会认为是用一个对象且不能进入队列中批量更新
      date: 2022/12/10
    - content: 好耶，马上就可以放假回家了！好想家里的好吃的😋！才不是想捏妹妹的脸了
      date: 2022/12/06
    - content: 全局音乐的动画也处理好了, nice!
      date: 2022/11/13
    - content: 把页脚, 首页顶部全都魔改到本地了, 方便后续魔改, 音乐也改成胶囊的样式了, 其实还是想让胶囊可拖拽, 不可点击改变歌词位置的, 但是弄了半天都没弄好就放弃了
      date: 2022/11/13
    - content: 朋友圈船新版本终于写完了, 耶✌️
      date: 2022/11/05
      link: https://hexo.anheyu.com/album/
    - content: 终于把相册集搞定了, 耶✌️, 瀑布流在滑动滚动条一个视口范围上下100的情况执行一次, 到底部停止监听让性能高了好多，再也不会布局混乱🤪了
      date: 2022/10/25
      link: https://hexo.anheyu.com/album/
    - content: 搜索🔍支持缩略图显示啦（默认获取文章内容的第一张图片）
      date: 2022/10/23 08:00:00
      from: 安知鱼
    - content: 遇见彩虹🌈吃定彩虹
      date: 2022/10/23 10:00:00
      image:
        - https://bu.dusays.com/2023/04/09/64329399e285d.webp
        - https://bu.dusays.com/2023/04/09/64329399aa3bc.webp
        - https://bu.dusays.com/2023/04/09/6432939996dd7.webp
    - content: ThreeJs API真多丫
      date: 2022/10/19
    - content: 妹妹强制要求我买走了她的两幅画 -¥30
      date: 2022/10/02
      image:
        - https://bu.dusays.com/2023/04/09/643293997b92b.jpeg
    - content: 歌曲推荐
      date: 2022/09/25
      aplayer:
        server: tencent
        id: 001FGQba3i10mw
    - content: 做了一个噩梦, 梦到从楼顶坠下去了。😷
      date: 2022/09/24
    - content: JOJO是真的好看！
      date: 2022/09/21
      link: https://www.bilibili.com/bangumi/play/ss39431?spm_id_from=333.337.0.0
```
主题配置文件中开启menu中关于和闲言碎语的注释，导航栏闲言碎语，注意缩进！！！
```yml
  关于:
    闲言碎语: /essay/ || icon-lightbulb
```
### 2.6.追番页面
在博客根目录执行
```bash
npm install hexo-bilibili-bangumi --save
```
在 hexo 配置文件_config.yml中加入以下配置，注意不是主题配置文件，更多配置请参考[hexo-bilibili-bangumi](https://github.com/HCLonely/hexo-bilibili-bangumi)
```yml
# 追番插件
# https://github.com/HCLonely/hexo-bilibili-bangumi
bangumi: # 追番设置
  enable: true
  source: bili
  path:
  vmid: 372204786
  title: "追番列表"
  quote: "生命不息，追番不止！"
  show: 1
  lazyload: false
  loading:
  showMyComment: false
  pagination: false
  metaColor:
  color:
  webp:
  progress:
  extraOrder:
  proxy:
    host: "代理host"
    port: "代理端口"
  extra_options:
    top_img: false
    lazyload:
      enable: false
```
图例如下
![alt text](https://img02.anheyu.com/adminuploads/1/2023/04/09/643264bec3298.png!blogimg)

### 2.7.朋友圈页面
在 Hexo 博客根目录 [blog]下打开终端，输入
```bash
hexo new page fcircle
```
打开[blog]\source\fcircle\index.md,添加一行**type: 'fcircle':**
```markdown
---
title: 朋友圈
date: 2022-11-21 17:06:17
comments: false
aside: false
top_img: false
type: "fcircle"
---
```
主题配置文件中开启menu中友链和朋友圈的注释，导航栏朋友圈，注意缩进！！！
```yml
  友链:
    朋友圈: /fcircle/ || icon-artstation
```
主题配置文件中开启friends_vue.enable，自行设置 朋友圈后端地址 和 顶部模块背景，注意缩进！！！
```yml
# 朋友圈配置
friends_vue:
  enable: false
  vue_js: https://npm.elemecdn.com/anzhiyu-theme-static@1.1.2/friends/index.f9a2b8d2.js
  apiurl: # 朋友圈后端地址
  top_background:
```
其中vue_js参数，可以将`https://npm.elemecdn.com/anzhiyu-theme-static@1.1.2/friends/index.f9a2b8d2.js`下载下来后将其中的 friends.anheyu.com替换为您的后端 url 然后上传至您的存储端以url的形式使用。
第二种办法也可以自行下载项目后，修改代码中的 url 变量路径friends.anheyu.com为你自己的，然后执行npm run build构建后将dist文件夹中的js上传至您的存储端使用
原项目地址：[hexo-circle-of-friends-front](https://github.com/anzhiyu-c/hexo-circle-of-friends-front/tree/anzhiyu)
> 注意朋友圈后端爬取需使用common2，否则无法爬取到您的友链数据。

图例如下：
![alt text](https://img02.anheyu.com/adminuploads/1/2023/04/09/64326468190c2.png!blogimg)

### 2.8.相册页面
#### 2.8.1.主页面
在 Hexo 博客根目录 [blog]下打开终端，输入
```bash
hexo new page album
```
你会找到 source/album/index.md 这个文件，修改这个文件： 记得添加 type: "album"
```markdown
---
title: 相册集
date: 2022-10-23 15:57:51
aside: false
top_img: false
type: "album"
---
```
主题配置文件中开启menu中我的和相册集的注释，**注意缩进！！！**
```yml
  我的:
    相册集: /album/ || icon-images
```
新建文件[blog]\source\_data\album.yml,没有_data文件夹的话也请自己新建。打开[blog]\source\_data\album.yml，输入：
```yml
- class_name: 世界各地夕阳与风景
  path_name: /wordScenery
  type: 2
  description: 因为到不了世界各地，所以请网友们发来了各地的夕阳与风景🌇。
  cover: https://upload-bbs.miyoushe.com/upload/2025/06/13/125766904/2cf2b6aea07bba089d0d17c4fea72d1b_5366629137934368264.png
  top_background: https://bu.dusays.com/2023/06/30/649e546ada7dd.webp
  rowHeight: 220
  limit: 10
  lazyload: true
  btnLazyload: false
  url: false
  top_link: /album
  top_btn_text: 返回
  album_list:
    - date: 2022/10/26 01:00:00
      content: 湘潭的一角。
      address: 湖南湘潭
      from: 再吃一口就减肥
      image:
        - https://bu.dusays.com/2023/04/09/64329399db122.webp
    - date: 2022-10-25
      content: 洛阳暴雨后的天空。
      address: 河南洛阳
      from: 紫菜卷
      image:
        - https://bu.dusays.com/2023/04/09/64329399db122.webp
        - https://bu.dusays.com/2023/04/09/64329399db2e1.webp

- class_name: 我的日常
  path_name: /dailyPhoto
  type: 1
  description: 这里存放的是有关我自己的一些沙雕生活与有趣的事情。
  top_link: /album
  top_btn_text: 返回
  top_background: https://bu.dusays.com/2023/04/09/64329399cea5a.webp
  cover: https://bu.dusays.com/2023/04/09/64329399cea5a.webp
  album_list:
    - date: 2022-10-24
      content: 老妹的画
      image:
        - https://bu.dusays.com/2023/04/09/643293997b92b.jpeg
```

#### 2.8.2.分页面
由于相册页面需要很多的 page，所以在写数据的时候自行写入路径path_name，示例数据中有两个path_name，所以需要再创建两个页面
注意新建的页面必须与path_name一致。
```bash
hexo new page dailyPhoto
hexo new page wordScenery
```
你会找到 source/dailyPhoto/index.md 和source/wordScenery/index.md两个文件，这两个为相册集详情页
然后内容为以下内容, 需在详情页加上type: "album_detail"
```markdown
---
title: 日常生活
date: 2022-10-23 15:57:51
aside: false
top_img: false
type: "album_detail"
---
```

```markdown
---
title: 世界各地风景
date: 2022-10-23 15:57:51
aside: false
top_img: false
type: "album_detail"
---
```
远程加载json示例数据
```json
[
  {
    "url": "https://cdn.jsdelivr.net/gh/jerryc127/CDN/img/IMG_0556.jpg",
    "alt": "IMG_0556.jpg",
    "title": "这是title"
  },
  {
    "url": "https://cdn.jsdelivr.net/gh/jerryc127/CDN/img/IMG_0472.jpg",
    "alt": "IMG_0472.jpg"
  },
  {
    "url": "https://cdn.jsdelivr.net/gh/jerryc127/CDN/img/IMG_0453.jpg",
    "alt": ""
  },
  {
    "url": "https://cdn.jsdelivr.net/gh/jerryc127/CDN/img/IMG_0931.jpg",
    "alt": ""
  }
]
```
示例图如下：
![alt text](https://img02.anheyu.com/adminuploads/1/2023/04/09/64326458a0f01.png!blogimg)
![alt text](https://img02.anheyu.com/adminuploads/1/2023/04/19/643f4351c8245.webp!blogimg)
![alt text](https://img02.anheyu.com/adminuploads/1/2023/04/19/643f42162d2f4.webp!blogimg)

### 2.9.音乐馆页面
在 Hexo 博客根目录 [blog]下打开终端，输入
```bash
hexo new page music
```
你会找到 source/music/index.md 这个文件，修改这个文件： 记得添加 type: "music"
```markdown
---
title: 音乐馆
date: 2021-04-24 21:41:30
type: music
aplayer: true
top_img: false
comments: false
aside: false
---
```
新建 source/json/music.json，此 json 为切换歌单按钮的歌单数据。
```json
[
  {
    "name": "青花瓷",
    "artist": "周杰伦",
    "url": "https://npm.elemecdn.com/anzhiyu-music@1.0.4/青花瓷/青花瓷.mp3",
    "cover": "https://y.qq.com/music/photo_new/T002R300x300M000002eFUFm2XYZ7z_2.jpg?max_age=2592000",
    "lrc": "https://npm.elemecdn.com/anzhiyu-music@1.0.4/青花瓷/青花瓷.lrc"
  },
  {
    "name": "稻香",
    "artist": "周杰伦",
    "url": "https://npm.elemecdn.com/anzhiyu-music@1.0.1/周杰伦/稻香/稻香.mp3",
    "cover": "https://y.qq.com/music/photo_new/T002R300x300M000002Neh8l0uciQZ_1.jpg?max_age=2592000",
    "lrc": "https://npm.elemecdn.com/anzhiyu-music@1.0.1/周杰伦/稻香/稻香.lrc"
  },
  {
    "name": "晴天",
    "artist": "周杰伦",
    "url": "https://npm.elemecdn.com/anzhiyu-music@1.0.2/晴天/晴天.mp3",
    "cover": "https://y.qq.com/music/photo_new/T002R300x300M000000MkMni19ClKG_3.jpg?max_age=2592000",
    "lrc": "https://npm.elemecdn.com/anzhiyu-music@1.0.2/晴天/晴天.lrc"
  },
  {
    "name": "七里香",
    "artist": "周杰伦",
    "url": "https://npm.elemecdn.com/anzhiyu-music@1.0.2/七里香/七里香.mp3",
    "cover": "https://y.qq.com/music/photo_new/T002R300x300M000003DFRzD192KKD_1.jpg?max_age=2592000",
    "lrc": "https://npm.elemecdn.com/anzhiyu-music@1.0.2/七里香/七里香.lrc"
  },
  {
    "name": "花海",
    "artist": "周杰伦",
    "url": "https://npm.elemecdn.com/anzhiyu-music-jay@1.0.1/花海/花海.flac",
    "cover": "https://y.qq.com/music/photo_new/T002R300x300M000002Neh8l0uciQZ_1.jpg?max_age=2592000",
    "lrc": "https://npm.elemecdn.com/anzhiyu-music-jay@1.0.1/花海/花海.lrc"
  },
  {
    "name": "反方向的钟",
    "artist": "周杰伦",
    "url": "https://npm.elemecdn.com/anzhiyu-music-jay@1.0.1/反方向的钟/反方向的钟.flac",
    "cover": "https://y.qq.com/music/photo_new/T002R300x300M000000f01724fd7TH_1.jpg?max_age=2592000",
    "lrc": "https://npm.elemecdn.com/anzhiyu-music-jay@1.0.1/反方向的钟/反方向的钟.lrc"
  },
  {
    "name": "兰亭序",
    "artist": "周杰伦",
    "url": "https://npm.elemecdn.com/anzhiyu-music@1.0.1/周杰伦/兰亭序/兰亭序.mp3",
    "cover": "https://y.qq.com/music/photo_new/T002R300x300M000002Neh8l0uciQZ_1.jpg?max_age=2592000",
    "lrc": "https://npm.elemecdn.com/anzhiyu-music@1.0.1/周杰伦/兰亭序/兰亭序.lrc"
  },
  {
    "name": "说好的辛福呢",
    "artist": "周杰伦",
    "url": "https://npm.elemecdn.com/anzhiyu-music@1.0.2/说好的辛福呢/说好的辛福呢.mp3",
    "cover": "https://y.qq.com/music/photo_new/T002R300x300M000002Neh8l0uciQZ_1.jpg?max_age=2592000",
    "lrc": "https://npm.elemecdn.com/anzhiyu-music@1.0.2/说好的辛福呢/说好的幸福呢.lrc"
  },
  {
    "name": "等你下课 (with 杨瑞代)",
    "artist": "周杰伦",
    "url": "https://npm.elemecdn.com/anzhiyu-music@1.0.1/周杰伦/等你下课/等你下课.mp3",
    "cover": "https://y.qq.com/music/photo_new/T002R300x300M000003bSL0v4bpKAx_1.jpg?max_age=2592000",
    "lrc": "https://npm.elemecdn.com/anzhiyu-music@1.0.1/周杰伦/等你下课/等你下课.lrc"
  },
  {
    "name": "我落泪情绪零碎",
    "artist": "周杰伦",
    "url": "https://npm.elemecdn.com/anzhiyu-music@1.0.2/我落泪情绪零碎/我落泪情绪零碎.mp3",
    "cover": "https://y.qq.com/music/photo_new/T002R300x300M000000bviBl4FjTpO_1.jpg?max_age=2592000",
    "lrc": "https://npm.elemecdn.com/anzhiyu-music@1.0.2/我落泪情绪零碎/我落泪情绪零碎.lrc"
  },
  {
    "name": "听妈妈的话",
    "artist": "周杰伦",
    "url": "https://npm.elemecdn.com/anzhiyu-music@1.0.2/听妈妈的话/听妈妈的话.mp3",
    "cover": "https://y.qq.com/music/photo_new/T002R300x300M000002jLGWe16Tf1H_1.jpg?max_age=2592000",
    "lrc": "https://npm.elemecdn.com/anzhiyu-music@1.0.2/听妈妈的话/听妈妈的话.lrc"
  },
  {
    "name": "明明就",
    "artist": "周杰伦",
    "url": "https://npm.elemecdn.com/anzhiyu-music-jay@1.0.1/明明就/明明就.flac",
    "cover": "https://y.qq.com/music/photo_new/T002R300x300M000003Ow85E3pnoqi_1.jpg?max_age=2592000",
    "lrc": "https://npm.elemecdn.com/anzhiyu-music-jay@1.0.1/明明就/明明就.lrc"
  },
  {
    "name": "我是如此相信",
    "artist": "周杰伦",
    "url": "https://npm.elemecdn.com/anzhiyu-music-jay@1.0.1/我是如此相信/我是如此相信.flac",
    "cover": "https://y.qq.com/music/photo_new/T002R300x300M000001hGx1Z0so1YX_1.jpg?max_age=2592000",
    "lrc": "https://npm.elemecdn.com/anzhiyu-music-jay@1.0.1/我是如此相信/我是如此相信.lrc"
  },
  {
    "name": "发如雪",
    "artist": "周杰伦",
    "url": "https://npm.elemecdn.com/anzhiyu-music@1.0.3/发如雪/发如雪.mp3",
    "cover": "https://y.qq.com/music/photo_new/T002R300x300M0000024bjiL2aocxT_3.jpg?max_age=2592000",
    "lrc": "https://npm.elemecdn.com/anzhiyu-music@1.0.3/发如雪/发如雪.lrc"
  },
  {
    "name": "以父之名",
    "artist": "周杰伦",
    "url": "https://npm.elemecdn.com/anzhiyu-music@1.0.3/以父之名/以父之名.mp3",
    "cover": "https://y.qq.com/music/photo_new/T002R300x300M000000MkMni19ClKG_3.jpg?max_age=2592000",
    "lrc": "https://npm.elemecdn.com/anzhiyu-music@1.0.3/以父之名/以父之名.lrc"
  },
  {
    "name": "园游会",
    "artist": "周杰伦",
    "url": "https://npm.elemecdn.com/anzhiyu-music@1.0.3/园游会/园游会.flac",
    "cover": "https://y.qq.com/music/photo_new/T002R300x300M000003DFRzD192KKD_1.jpg?max_age=2592000",
    "lrc": "https://npm.elemecdn.com/anzhiyu-music@1.0.3/园游会/园游会.lrc"
  },
  {
    "name": "本草纲目",
    "artist": "周杰伦",
    "url": "https://npm.elemecdn.com/anzhiyu-music@1.0.4/本草纲目/本草纲目.mp3",
    "cover": "https://y.qq.com/music/photo_new/T002R300x300M000002jLGWe16Tf1H_1.jpg?max_age=2592000",
    "lrc": "https://npm.elemecdn.com/anzhiyu-music@1.0.4/本草纲目/本草纲目.lrc"
  },
  {
    "name": "龙卷风",
    "artist": "周杰伦",
    "url": "https://npm.elemecdn.com/anzhiyu-music@1.0.4/龙卷风/龙卷风.mp3",
    "cover": "https://y.qq.com/music/photo_new/T002R300x300M000000f01724fd7TH_1.jpg?max_age=2592000",
    "lrc": "https://npm.elemecdn.com/anzhiyu-music@1.0.4/龙卷风/龙卷风.lrc"
  }
]
```
hexo 配置文件_config.yml中添加以下配置，**注意不是主题配置文件**
```yml
# APlayer
# https://github.com/MoePlayer/hexo-tag-aplayer/blob/master/docs/README-zh_cn.md
aplayer:
  meting: true
  asset_inject: false
```
主题配置文件中开启menu中我的和音乐馆的注释，**注意缩进！！！**
```yml
  我的:
    音乐馆: /music/ || icon-music
```

> 如何修改默认歌单?
将menu中音乐馆的路径修改为以下格式即可/music/?id=1708664797&server=tencent，支持id和server参数。
id 与 server 的填写请参考[MetingJS](https://github.com/metowolf/MetingJS)
图例：
![alt text](https://img02.anheyu.com/adminuploads/1/2023/04/09/643264b4da332.png!blogimg)

## 3.参考资源
[安知鱼主题文档](https://docs.anheyu.com/initall.html)

## 4.结语
到这里，Anzhiyu 主题绝大部分特色独立页面的配置就全部讲解完毕。
我们一共配置了友人帐、关于、我的装备、留言板、即刻短文、追番页、朋友圈、相册集、音乐馆九大页面。这些页面是 Anzhiyu 区别于普通 Hexo 主题的核心亮点，不用自己编写 HTML 与 CSS 模板，依靠主题内置渲染能力，只需要生成页面、维护`_data`目录下的数据源 yml、调整导航菜单，就可以快速搭建起功能丰富的个人博客站点。
配置完这些页面之后，你的博客已经不再只有单纯的文章发布功能，既可以展示个人信息、数码好物，也能够分享生活碎片、相册照片、爱好番剧与音乐，还可以和其他博主互换友链、接收访客留言，博客的完整度与个人风格直接拉满。
后续可以继续去完善评论系统、站点 SEO、站点统计等内容，进一步打磨博客体验。希望本篇教程能够帮到正在折腾 Hexo 博客的你。