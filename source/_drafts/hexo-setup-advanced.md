---
title: hexo-setup-advanced
tags:
---
### Hexo Setup



#### Remote repos

1. Create a repo for blog, URL will be like `git@github.com:miyunda/hexo-blog.git` , make a branch `blog-src` and set as default.

2. Fork https://github.com/theme-next/hexo-theme-next , this will be a sub-repo of blog repo. URL will be like `git@github.com:miyunda/hexo-theme-next.git` .
new a branch, name like `my-next` .

3. Generate an SSH key pair for blog repo and another one for theme repo.

```bash
ssh-keygen -t rsa -b 4096 -C "404@beijing2b.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/yu/.ssh/id_rsa): /Users/yu/.ssh/hexo-blog
```

```bash
ssh-keygen -t rsa -b 4096 -C "404@beijing2b.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/yu/.ssh/id_rsa): /Users/yu/.ssh/hexo-next
```

4. Copy contents of `~/.ssh/hexo-blog.pub` and paste into blog repo settings. Copy contents of `~/.ssh/hexo-next.pub` and paste into theme repo settings.
​    - [x] Allow write access

5. Config SSH agent

Edit `~/.ssh/config`, contents will be like

```
  Host next-github
​    HostName github.com
​    AddKeysToAgent no
​    UseKeychain yes
​    IdentityFile ~/.ssh/hexo-next

  Host blog-github
​    HostName github.com
​    AddKeysToAgent no
​    UseKeychain yes
​    IdentityFile ~/.ssh/hexo-blog
```

6. Test the SSH key. Below result is expect.
   ```bash
   ssh -T git@blog-github

   Hi miyunda/hexo-blog! You've successfully authenticated, but GitHub does not provide shell access.

   ssh -T git@next-github
   Hi miyunda/next-blog! You've successfully authenticated, but GitHub does not provide shell access.


7. Generate a personnal token, **write it down**.

---
#### local folder



1. Create local folder on WorkstationA

```bash
git clone -b blog-src git@blog-github:miyunda/hexo-blog.git
```

2. cd the  `hexo-blog` , Edit `.gitignore`, contents will be like

```
.DS_Store
.AppleDouble
.LSOverride
._*
Thumbs.db
db.json
*.log
*~
node_modules/
public/
.deploy*/
```

3. Clone theme to local
   ```bash
   git fetch
   git submodule  add -b my-next -f git@next-github:miyunda/hexo-theme-next.git themes/next

   ```



4. modify your settings, write something in `source/` as `md` files.



5. Commit the change
   ```bash

   git add .

   git commit -am "add blog source and theme"

   ```



6. Upload to repo



```bash

git push origin blog-src

```



#### CI/CD



1.  

2. Go to `Travis` CI



3. Edit `.travis.yml`, contents will be like

  ```yaml
   # 指定构建环境是Node.js，当前版本是稳定版
   anguage: node_js
   node_js: stable
   # 设置钩子只检测blog-src分支的push变动
   branches:
​     only:
​       - blog-src

   # 设置缓存文件
   cache:
​     directories:
​       - node_modules

   #在构建之前安装hexo环境和主题，这里的主题就是原来修改过的主题，我将其托管到另一个github仓库，直接clone就行，否则每次都是新的主题，要重新设置。

   before_install:
​     - npm install -g hexo-cli
​     - git clone https://github.com/miyunda/hexo-theme-next.git themes/next
 

   #安装git插件和搜索功能插件

   install:
​     - npm install
​     - npm install hexo-deployer-git --save
​     - npm install hexo-generator-searchdb --save

   # 执行清缓存，生成网页操作
   script:
​      - hexo generate

   

   # 设置git提交名，邮箱；替换真实token到_config.yml文件，最后depoy部署
  ``` 



4. Intstall command line tool
Skip this step if you have ruby >= 2
```bash
curl -sSL https://get.rvm.io | bash -s stable --ruby
source ~/.rvm/scripts/rvm
```

Install travis command line tool via rvm, enecyrpt your ssh key and store in your blog source repo.
```bash
gem install travis
travis login --pro # login with your GitHub cred
travis encrypt-file ~/.ssh/web-bj-01_id_rsa --add --com
```
5. commmit your change and push , you will see the project starts building in Travis web console. 

##### usage
1. When compeleted your new page/post, saved in md file.
```bash
git commit -m "some descripts" # in blog root folder
git push
```
2. When modified settings in theme next.
```bash
gegit commit -m "some descripts" # in blog themes/next folder
git push
cd ../..
git submodule update --remote
git add .
commit -m "record my-next SHA1 gitlink"
git push
```
