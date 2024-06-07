---
title: 搭建Hexo`个人博客`
date: 2024-06-07 16:34:41
tags:  基础
---

#### 使用Hexo

#### 使用Hexo

##### 使用前需要前置安装软件

​	1 、node

​	2 、npm

​	3 、git 

##### 搭建仓库

1. 使用github 创建新仓库，仓库名最好是github用户名+github+io
2. 选public 
3. add a README file   然后创建
4. 配置ssh keys方便提交代码

```
进入任意文件夹使用git bash 输入
ssh-keygen -t rsa -C "邮件地址"
```

敲4次回车，然后进入C:\user\\用户  文件夹中找到.ssh 文件，使用文本编辑器打开id_ras.pub复制代码信息。

在GitHub中打开用户设置，找到SSH keys 新增一个SSH， 将生成的密钥粘贴进去保存。

在任意文件夹中打开git bash  输入 

```
 ssh -T git@github.com  
 询问时输入yes  出现successfully表示添加完成
```

 检查仓库信息

```
查看当前远程仓库的名称
git remote -v
如果没有仓库信息则需要添加仓库信息
添加仓库
git remote add origin  仓库地址信息可以时http 或者ssh

提交代码 在根目录下打开git bash
git add .   //添加当前目录以及子目的到git

git commit -m "填写提交说明信息"

推送到远程仓库

git push -u origin master
```



##### 下载Hexo

​	1 nmp   install  hexo-cli -g

##### 创建Hexo文件

​	1、找到要创建的目录打开命令行，输入 hexo init  my-blog      # my-blog 为创建的博客文件名

​	 2、cd my-blog  进入刚创建的文件中，npm  install 安装一下，

​	 3、配置git 部署插件 ： npm install hexo-deployer-git --save

​	 4、配置"_config.yml" 文件 在github上使用，打开"_config.yml"文件，在尾部修改数据

```
deploy:
  type: git
  repo: 复制github 刚创建仓库ssh地址信息
  branch: master
```

#### 部署hexo

5、生成静态文件并部署

​	 生成静态：hexo  generate   部署hexo deploy 

​	 可以合并成一步 hexo g -d  

​	6、查看博客信息

​		https://xxxx.github.io

#### 使用 Butterfly 主题 

1. 安装 Butterfly 主题 

		进入项目根目录中拉去主题信息

```bash
git clone https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly
```

2. 配置 Butterfly 主题 

   在 Hexo 项目的 `_config.yml` 文件中配置 Butterfly 主题。编辑 `_config.yml` 文件，将 `theme` 字段设置为 `butterfly`： 

3. 安装插件

   ```nmp
   npm install hexo-renderer-pug hexo-renderer-stylus --save
   ```

4. 配置yml

   在hexo项目根目录下创建文件_config.butterfly.yml ，将主题Butterfly目录中 _config.yml文件内容复制到 _config.butterfly.yml 文件中。

   #### 创建新文章并发布

   1.进入项目目录执行命令

   ```bash
   hexo new "文章名称"
   
   这将在 source/_posts/ 目录下创建一个新的 Markdown 文件，文件名格式为 YYYY-MM-DD-my-new-post.md，其中 YYYY-MM-DD 是创建日期，my-new-post 是你指定的文章标题。
   ```

   2.使用编辑工具打开文章进行编辑

   3.生成静态文件，使用编辑器编辑完后，使用命令生成静态信息

   ```bash
   hexo generate
   ```

   4.使用git 命令推送到远端仓库

   5.使用部署功能 

   ```bash
   hexo deploy
   ```

   6.总结新文章部署

   ```
   # 生成静态文件
   hexo generate
   
   # 提交更改到 GitHub
   git add .
   git commit -m "提交说明"
   
   git push origin master
   或者
   git push origin main
   
   # 部署到 GitHub Pages
   hexo deploy
   ```

   





