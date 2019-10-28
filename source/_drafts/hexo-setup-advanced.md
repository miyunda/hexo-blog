---
title: 利用CI/CD部署Hexo网站
tags:
---





#### Remote repos

1. Create a repo for blog, URL will be like `git@github.com:miyunda/hexo-blog.git` , make a branch `blog-src` and set as default.

2. Fork [theme Next](https://github.com/theme-next/hexo-theme-next) , this will be a sub-repo of blog repo. new URL will be like `git@github.com:miyunda/hexo-theme-next.git` .
 We will work on the default branch `master` .

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

3. Clone the theme Mext to local
   ```bash
   git fetch
   git submodule  add -f git@next-github:miyunda/hexo-theme-next.git themes/next

   ```



4. Write something into a new page/post.
copy contents from `themes/next/_config.yml` to your blog root `_config.yml`, also make two-blankspace-indentation at very beginning of each lines pasted.
add a line of `theme_config:` on top of them. By this way, themse settings are seperated from themes files.
5. config user identity.
```bash
git config user.name "blog source  updater"
git config user.email "hexo-blog-updater@beijing2b.com"
```
  also config user identity for theme Next.
  ```bash
git config user.name "blog theme updater"
git config user.email "hexo-blog-updater@beijing2b.com"
```
6. Commit the change
   ```bash
   git add .
   git commit -am "init blog source and theme"
   ```



6. Push to GitHub
```bash
git push origin blog-src
```
---
#### CI/CD

1.  

2. Go to `Travis` CI



3. create file `.travis.yml`
```bash
touch .travis.yml
```

4. Intstall command line tool
Skip this step if you have ruby.
```bash
curl -sSL https://get.rvm.io | bash -s stable --ruby
source ~/.rvm/scripts/rvm
```

Install travis command line tool via rvm, enecyrpt your ssh key and store in your blog source repo.
```bash
gem install travis
travis login --pro # login with your GitHub cred
travis encrypt-file ~/.ssh/web-bj-01_id_rsa --add --com
mv web-bj-01_id_rsa.enc .travis
```
```bash
```travis encrypt "beijing2b:YOURSLACKTOKEN" --add notifications.slack
Edit `.travis.yml`, contents will be like
```yaml
notifications:
  slack:
    secure: MF1R4Da7eIrZuHmp6EMQHcXiqnKrETxQBUYNj92XA3mQS7pl5H14K1APv9Tajmefj5SnIldJGyyFyvIu+aH6HMzvqyiRvSkitR81epZXNLb1PRRxtzAgo7aVejRFCHpPIm1dWD6zsXlX7rdcqYvJ1NRb/OpKuwatF3MF0Fxu2p6Z6VjYmZJhdd5FBrCTmLXzDRrJOkZk2AlMOZkvgQKvEEMaCCwt1PqcoXbf9fCov4jmEVj66imJW5V2SvY6omfG6LFTugA4DhOyRq/YWWdLkiWMFibOpk0zt4jrb5gu7dnKV2Jwi7/K3QRZhdiA29+c7nh4kQTPWvcr1WAwEIFqqw6ERSFVWcaUCjNenLO1AG41SQizg5EymkYQNhXJf9WuigHmQpcQViz0O0Hvy0i3s/iKMt5IX+aWUdrhSHYM2MITptdtInGiRSN7kik1DLaZIuztnliV9wisWqNRDf18LSfDNvU5OtBIr9RvEaWNUJYyoELul1aXcdqCR4dXynLdc4FnhLCCa6EayoeWxq77oqUTBAmxlbHkTUD7O1vj3VvfOj0CgPP05cO5rJhj1DNiy69D3Zk8T2N8ZdP6sV10Bdy7AB8jtRkz3xb4Qdei2e2BmS+2Fx3a9XHTxG8Q04Z828DCLGnQZCVfb645EQR6RuTFwhsAjR/q4rG7aWdYHzQ=
# 不克隆子仓库
git:
  submodules: false
# 设置钩子只检测blog-src分支的push变动
branches:
  only:
  - blog-src
# 设置缓存文件
cache:
  directories:
  - node_modules
#在构建之前安装ssh环境，hexo环境和主题。
before_install:
- openssl aes-256-cbc -K $encrypted_f1eb9b6ab60a_key -iv $encrypted_f1eb9b6ab60a_iv -in .travis/web-bj-01_id_rsa.enc -out ~/.ssh/web-bj-01_id_rsa -d
- chmod 600 ~/.ssh/web-bj-01_id_rsa
- eval $(ssh-agent)
- ssh-add ~/.ssh/web-bj-01_id_rsa
- cp .travis/ssh_config ~/.ssh/config
- npm install -g hexo-cli
- rm -rf themes/next
- git clone  https://github.com/miyunda/hexo-theme-next.git themes/next
#安装需要的插件
install:
- npm install
- npm install hexo-deployer-git --save
- npm install hexo-generator-searchdb --save
- npm install hexo-tag-dplayer --save
#生成html静态文件
script:
- hexo generate
#部署到web服务器
after_script:
- hexo deploy
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
git commit -m "some descripts" # in blog themes/next folder
git push
cd ../..
git submodule update --remote
git add .
commit -m "record my-next SHA1 gitlink"
git push
```