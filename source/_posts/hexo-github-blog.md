---
title: Hexo+github一步一步搭建个人博客
tags: [博客,hexo,github]
toc: true
reward: true
---

<p>最近用使用Hexo和github搭建了一个个人博客，现在记录下来;
Hexo是一个强大的博客框架,这里是[中文文档](https://hexo.io/zh-cn/docs/index.html)</p>

### **一、基础博客搭建流程**
#### 安装Node和Git
  - windows：
    下载Node.js客户端安装即可。
    在命令行输入node -v出现如下图所示就安装成功了
    ![Alt   text](http://oqnan33k8.bkt.clouddn.com/myblog_img/hexo_github_blog/node.png)
<!-- more -->
  - 下载git(国内直接从官网下载比较困难，需要翻墙。这里提供一个国内的下载站)[download](https://github.com/waylau/git-for-win)
  安装正确后 在桌面或文件夹空白处鼠标右键菜单会新增“Git GUI Here”和“Git Bash Here”两个选项。

#### 使用hexo搭建博客
 - **安装全局hexo**
   右键运行Git Bash 执行`npm install -g hexo`;安装后输入`hexo -v`,出现版本信息表示安装成功。
 - **在项目中安装hexo**
    新建文件夹为你的博客项目名，进入项目打开Git Bash
    `npm install hexo --save`在项目中安装hexo;
    `hexo init`初始化hexo
    `npm install`安装hexo所需依赖包
 - **安装hexo插件**
    `npm install hexo-server --save` 本地服务所需插件
    `npm install hexo-deployer-git --save`使用git进行部署所需插件
#### 在本地生成博客静态页面并预览
在项目路径下打开Git Bash
 - **在本地生成静态页面**
  `hexo generate`,生成一个存放静态文件的文件夹public;该命令可以简写为:`hexo g`
- **启动本地服务器**
  `hexo server`,启动服务;简写为`hexo s`;
  默认网址为：`http://localhost:4000/`
  默认端口为4000，如果端口被占用,执行`hexo s -p 5000`表示指定服务端口为5000。

如果以上步骤都不出意外的话，你就会看到一个Hexo博客初始化的页面。

### **二、GitHub+hexo配置个人博客**
 **上面介绍了如何在本地搭建博客，接下来开始配置GitHub并关联Hexo**
 - **GitHub pages**
1. 首先注册一个GigHub帐号，注册比较简单就不再赘
2. 帐号创建号后，需要创建一个仓库(Respository);**注意:**</b>仓库名字要与GitHub用户名一致,比如我的用户名是FireYao,创建的respository名就是FireYao.github.io
3. 创建好respository后，进入到该respository界面,点击settings拉到最下方找到GitHub Pages,点击 **Launch automatic page generator**,让GitHub生成GitHubPager

- **配置SSH Keys**
  1. 本地生成ssh密钥。
  git bash下输入`ssh-keygen -t rsa -C ‘你的邮箱地址’`
  2. 上传本地的公钥串，使当前电脑与GitHub账户建立联系。
  3. 在你的电脑C：\ Users\你的计算机用户名.ssh目录下打开刚刚生成的id_rsa.pub，复制里面的内容。然后点击你GitHub账户右上角的头像，选择settings，找到SSH and GPG keys，点击进入之后再点击New SSH key，title随便写，把公钥串粘贴到文本框，保存即可
- **在Hexo配置文件中关联GitHub账号**
  1. 在之前搭建好的本地博客项目中编辑` _config.yml`文件,把其中的deploy参数（没有的话就按如下格式新建，注意冒号后面一定要有一个空格），修改为：
  ```code
  deploy:
  	type: git
  	repo: https://github.com/FireYao/FireYao.github.io.git
  	branch: master
  ```
  2. 重新部署项目
    在博客根目录打开Git Bash依次执行
    ```code
    hexo clean    #会清除缓存文件db.json及之前生成的静态文件夹public；
    hexo g     #会重新生成静态文件夹public；
    hexo deploy    #把本地生成的静态文件部署到FireYao.github.io这个仓库中的master分支上；简写形式为hexo d
    ```
    `hexo g 和 hexo d可以合并在一起写：hexo g -d`
  3. 在浏览器中访问博客
     在浏览器中输入`FireYao.github.io`,没毛病的话，你应该就能看到之前在本地搭建的那个博客页面了。
### 小结    
<p>
  到这里已经通过Hexo创建了一个最原始的博客，并将博客的静态文件存放到github仓库中，通过外网以github的默认域名访问这个博客。
</p>
**未完待续**
