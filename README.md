# GitHub+Hexo 搭建个人网站

## 前期准备

1. GitHub创建个人仓库

   点击GitHub中的New repository创建新仓库，仓库名应该为：**你的github用户名.github.io**，这是固定写法

2. 安装Git

   安装成功后，设置user.name和user.email

   ```yml
   git config --global user.name "你的GitHub用户名"
   git config --global user.email "你的GitHub注册邮箱"
   ```

   生成ssh密钥文件：

   ```bash
   ssh-keygen -t rsa -C "你的GitHub注册邮箱"
   ```

   然后直接三个回车即可，默认不需要设置密码
   然后找到生成的~/.ssh的文件夹中的id_rsa.pub密钥，进入刚刚建立的github仓库，打开GitHub_Settings_keys页面，新建new SSH Key，将内容全部复制。

   在Git Bash中检测GitHub公钥设置是否成功，输入 ssh git@github.com ：

   > 这里之所以设置GitHub密钥原因是，通过**非对称加密的公钥与私钥**来完成加密，公钥放置在GitHub上，私钥放置在自己的电脑里。GitHub要求每次推送代码都是合法用户，所以每次推送都需要输入账号密码验证推送用户是否是合法用户，为了省去每次输入密码的步骤，采用了ssh，当你推送的时候，git就会匹配你的私钥跟GitHub上面的公钥是否是配对的，若是匹配就认为你是合法用户，则允许推送。这样可以保证每次的推送都是正确合法的。

3. 安装Node.js

   注意安装Node.js会包含环境变量及npm的安装，

   安装后，检测Node.js是否安装成功，在命令行中输入 node -v :

   检测npm是否安装成功，在命令行中输入npm -v :

4. 安装Hexo

   使用npm命令安装Hexo，输入：

   ```bash
   npm install -g hexo-cli 
   ```

   安装完成后，进入你想存放博客的目录下，初始化博客，输入：

   ```bash
   hexo init blog
   ```

   为了检测我们的网站是否完成初步搭建，分别按顺序输入以下三条命令：

   ```bash
   hexo new test_my_site
   hexo g
   hexo s
   ```

   完成后，打开浏览器输入地址：localhost:4000即可看到博客搭建完成

## 配置文件说明

- blog根目录里的_config.yml文件称为**站点**配置文件

- 进入根目录里的themes文件夹，里面也有个_config.yml文件，这个称为**主题**配置文件

> 如果需要修改博客的主题，则需要下载相应的theme，并在站点配置文件修改theme

## 相关命令解析

- 创建新博客

    ```BASH
    $ hexo new "My New Post"
    ```

    这时会在`source/_posts`文件夹内出现一个新的My New Post.md，编写即可

- 保存提交部署

  ```bash
  $ hexo clean
  $ hexo generate
  $ hexo deploy
  $ hexo clean && hexo generate && hexo deploy
  或者，简写
  $ hexo clean && hexo g && hexo d
  ```

  命令解释：

  - **hexo clean**

    清除缓存文件 (`db.json`) 和已生成的静态文件 (`public`)。

    在某些情况（尤其是更换主题后），如果发现对站点的更改无论如何也不生效，可能需要运行该命令。

  - **hexo generate**

    生成静态文件

  - **hexo deploy**

    部署网站，部署完成后即可使用**https://你的github用户名.github.io**进行博客访问

- 本地启动

  ```bash
  $ hexo server
  $ hexo s
  然后通过http://localhost:4000/查看
  ```
  
- git提交

  ```bash
  git add
  git commit -m 
  git push origin hexo(hexo分支是源代码，master分支是部署网页静态页面的)
  ```


## 文章内容编写

- 文章使用markdown语言编写，具体放在`source/_posts`文件夹内


- 开头的标签、分类、置顶如下：

> title: My New Post
> date: 2019-10-12 21:56:33
> categories: life
> tags: life
> top: true    //默认false

- 多个tags或者categories
  
```
  tags: [life, 刷题]
  
  或者
  
  tags:
  - 日志
  - java
  - MyBatis
```

- 图片

  图片放在./source/img里面，可以这样引用

    ```html
  <div align="center">
      <img src="https://Maggie720.github.io/img/test.jpg" width="200px" height="300px">
      <img src="https://Maggie720.github.io/img/test.jpg" width="200px" height="300px">
  </div>   
  // 这个需要把图片上传到github之后才能看到，即完成hexo deploy命令之后
  
  或者
  
  ![](/img/test.jpg)  // 这个在本地md文件中看不到图片
    ```
  
- 阅读全文：

  在相关位置增加`<!--more-->`

  
