---
layout: post
title: 我花9块钱搭了一个配合个人博客使用的个人图床
date: 2019-8-8
tag: GEEK
---

## 背景
之前一直把简书当做自己的私人“图床”来用，因为简书的Markdown编辑器非常稳定（~~对比少数派的而言~~），实时保存加图片ctrl+v复制进去就能用的特征让我一直坚持在简书上首发自己写的东西，然后图方便的就直接把md的源文本直接copy到自己的博客站去发布，就省去了再给自己博客站里面文章增加一次配图的麻烦。但是简书也有自己的限制，就是会莫名其妙和谐你一堆文章，然而我在里面更新大多都是一些周记和月总结之类的文章，所以也让我很是苦恼，自己的博客站是用基于 Jekyll 和腾讯开发者平台 Cloud-studio和coding仓库做的，虽然访问比较慢，但是确实是真的全免费的（相当于GitHub里面的page功能，这个还可以绑定自己的[个人博客域名“blog.deghao.org”](blog.denghao.org)）。但是每次上传图片到代码仓库文件夹在到编辑器里面去手动复制输入如下
>`![时间块](https://upload-images.jianshu.io/upload_images/10043074-15cd21c43dfc8837.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)`

这么长一段的东西看起来也太不极客了，所以今天我走了上摸索搭建一个自己专用图床之路。

## 1.图床知识普及
1. 什么是图床

一般的咱们在网上看到的文章里的插图，其实在浏览器上就是一个图片链接，那个链接所指向的服务器就是所谓的“图床”。粗暴的理解就是文章插图所存放的地方，图床上的图片都可以一串地址链接的形式被用在网络里的各个地方。
截图中简书里自带的图床的地址。可以再浏览器的前端各种应用样式，但源地址始终都是那个“=”等号后面的“http://XXX/XXX/XXX.png”，那就是图床地址。
![图床说明](http://pic.denghao.org/pic/20190808161603.png)

图床地址还有个最简单识别的特征，就是你复制图片的地址，粘贴到浏览器里面是可以直接打开图片的。这个地址的前缀就是简书的图床服务器地址及文件绝对路径了。

![](http://pic.denghao.org/pic/20190808161859.png)

2. 为什么需要用图床

不知道你们是不是都经历过把QQ空间图片复制到贴吧，发布之后显示QQ空间图片禁止引用，或者把百度图片复制到微信公账号，发布之后会变成百度图片禁止使用之类的警告的经历。
![](https://pic.denghao.org/pic/20190808192718.png)

我相信只要在网上写过文章的人应该都会有遇到过上图情况，最后只能没辙，只能把想要用的图片先下载到本地再上传插入到要写的文章里面，这样显示就正常没问题了。

背后的原因也很简单，各大网站的图床都限制了只能自家的网站访问，你把图床的地址复制到竞争对手那里去了，版权问题到不说，就给他服务器带来的流量压力就增大了。
咱们每一次在网页上访问查看一次图片就是在经历图片从图床下载到你本地浏览器的一次过程，也就是在占用服务器的宽带资源（也就是钱），当然谁家都不愿意花着自家的钱，给其他家平台上的文章做添彩的嫁衣了，所以各家都会在技术层面限制被其他网站直接把图床作为外链插入使用。所以咱们需要拐一道弯：先把图片右键下载到本地再上传到目标站这么麻烦，其实底层原理就是换到你想发文章或图片的那家的图床服务器了。
现在好一点的文章编辑器，例如简书，他就可以根据你复制进去的图片的外链地址自动把该图片下载但他们服务器上生成新的属于简书自己的图床地址，大大的帮作者提升写文章配图的效率。但是最近估计简书也意识到不限制外链给服务器也带来压力了所以也开始现在外链使用了。我的个人博客里的插图全都变成了下面这个样。
![博客图片现状](http://pic.denghao.org/pic/20190808163119.png)

3. 搭建自己的图床有啥好处
~~可以配合自己的域名使用来装逼了~~
- 不用担心自己的辛辛苦苦截图写的文章因为被和谐，网站关闭啥的就丢了找不到了。
- 写自己的博客不需要像之前那么折腾了。

## 2.搭建自己图床的过程
### 1.买一个OSS对象存储服务
通俗的解释这就是一块云盘，只不过他可以通过接口api的形式去使用，面向的对象是开发人员而不是咱们日常客户。主要应用场景就是在存取非结构性数据文件上，就是不是那种数据库里存的结构化数据（目的就是区分云数据库的功能）。咱们就存些文章插图，40G足够用啦，不够到时候再升。五年和一年的单价一样都是9块一年并没有便宜那咱们就续费一年好了，谁说36块钱不算现金流了。
![收费](http://pic.denghao.org/pic/20190808165620.png)
购买完成之后还需要点击一下开通OSS服务功能。然后就可以进到OSS控制台了。
对了关于流量包购买的问题，不开通流量包就是默认为按量收费，大概算了下1.2元10个G每月，还行吧不是很贵，用着再说。
![计费标准](https://pic.denghao.org/pic/20190808174003.png)

### 2. 创建一个Bucket
这个bucket就是顾名思义的桶，你买了40G的空间可以分很多个桶，根据你对想存的东西的定义来划分。
像咱们做图床，访问频率是不会很低的，如果选低频甚至归档，每次看个你的文章插图需要1分钟冷启动，那黄花菜都凉了，就选择标准存储类型及公共读，加密那个也不用了，都为的是让所有人都看所以肯定选择公共度及不加密了。
![创建bucket](http://pic.denghao.org/pic/20190808170216.png)

### 3. 绑定自己的域名及CDN加速
没有自己域名，或者有域名没备案的小伙伴可以不用看这部分。
- 1.绑定自己名下的域名

我看了一下是可以直接绑定一级域名的，但是我denghao.org的一级用在个人主页了，就新解析了一个pic.denghao.org
![绑定域名](http://pic.denghao.org/pic/20190808170909.png)

- 2. 配置CDN及HTTPS证书

添加完成自己的域名之后发现居然还可以使用CDN加速，就点击配置，按要求下一步，然后再CDN中就有了这个新解析的2级域名，但是有个红点提醒要进行手动进行CNAME解析，继续就按要求完成CNAME解析，这样每次访问我的图片就可以通过CDN加速访问了，所以在当初购买OSS时候选择的服务器地址在北京就没啥重要的了，常访问的文件都在CDN里直接帮咱提高了访问速度。
然后就是HTTPS证书，CDN都弄了不再加个证书怎么行，直接提交申请免费的，绑定就能用了。因为都是一次性到位弄好的，没来得及截图，相信在仔细看这篇文章跟着实操的小伙伴应该能搞定的。

![HTTPS及CDN](https://pic.denghao.org/pic/20190808172715.png)

【CDN小知识】把常访问的文件放置在就近阿里全国各地地的服务器节点上，提高用户体验，简单说明就是我虽然把文件存在北京的服务器OSS上，但是海南的用户实际访问到的是CDN把我的常用（常被访问图片）临时的放置的广东或者深圳的CDM服务器节点了。这是一个很流行的提高用户前端浏览效率的工具服务，按流量收费也要不了多少钱的~~（我对自己的博客很自信）~~。

![域名指向拓扑图](https://pic.denghao.org/pic/20190808171732.png)
就是我的域名先指向CDN给我分配的一个专用地址，然后那个CDN专用地址再指向了OSS的源bucket专用地址，实现了一个数据流的更快更安全的访问我传的图片。

### 4. 配置和使用PicGo
这是个开源软件，可以再GitHub找到源码的，我从github上下的window安装包。“闪电”般的速度下载了40分钟。
- keyid和keysecret 这个是阿里账户专用的一个密钥，从头像这里找到，没有就创建一个，有就自己回忆回忆存哪了，或者新建一个子密钥
![密钥](https://pic.denghao.org/pic/20190808174256.png)
- 空间名就是bucket的名字
- 储存区域就是你当时选的服务器地区的编码从控制台可以找到
![](https://pic.denghao.org/pic/20190808174545.png)

- 最后的自定义域名就是你在用你图床外链的时候要显示的域名，可以用默认阿里给你分配的，也可以用自己绑定的自己的个人域名，我就用了自己的解析的pic.dengho.org，加上我安装了https证书，所以前面还能加个S。

【HTTPS小贴士】尽量上个证书加一下这个不起眼的S，现在微信对不是https的域名使用都限制了访问，支持更安全的https访问是主流趋势，偷一次懒之后又全得补上我都不敢想象有多麻烦。

![PICGO配置](https://pic.denghao.org/pic/20190808173628.png)

- 使用也非常简单，上传就是拖拽和剪切板上传2种，我试了一下可以一次拖拽多张图片上传，剪切板上传除了用电脑截图的方式、也同样可以复制其他网站上的（例如百度图片、微博图片）直接粘贴上传。

![](https://pic.denghao.org/pic/20190808174819.png)

- 上传成功的图片会在图床相册里，只有点击一下想要用的图片的左下角的复制按钮就能直接变成现成的Markdown图床链接样式，就可以直接愉快的插入MD编辑器中使用了，像下面这样。
更多的操作方法可以直接查看[PicGo操作手册](https://picgo.github.io/PicGo-Doc/zh/guide/#%E5%BA%94%E7%94%A8%E8%AF%B4%E6%98%8E"%3Ehttps://picgo.github.io/PicGo-Doc/zh/guide/#%E5%BA%94%E7%94%A8%E8%AF%B4%E6%98%8E)。

![](https://pic.denghao.org/pic/20190808175400.png)

## 3. 新图床的撰文及发布工作流

1. 印象笔记Markdown的编辑器直接码字
2. 写完的直接通过博客后台新增md文件把博文粘进去
![](https://pic.denghao.org/pic/20190808175832.png)
3. 提交更新，推送更新（git add. & git push）
4. 把博文md代码复制到简书，点击发布会自动化简书本地化图床地址（省图床流量至关重要的一招）
5. 用自媒体工具把简书上的文章群发到所有自媒体平台头X号之类的自媒体平台，更新上传图片都是不需要从我图床地址读图了，现在明白我为啥敢这么大方的全用按量收费了吧。
![](https://pic.denghao.org/pic/20190808191952.png)

大家有其他关于搭建图床的问题都可以私信或留言与我交流。
