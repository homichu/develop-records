# 开发记录（node.js+nuxt+mongodb）
## 服务器购买
  Heroku 是一个支持多种编程语言的云服务平台，Heroku 也提供免费的基础套餐供开发者测试使用。但是新版的Heroku需要信用卡，需要注意。
  还是趁打折买阿里云方便，就是贵，之后直接在阿里云买域名，管理方便。
  现在买阿里云ecs好像已经没了经典网络，都是直接专有网络，还是遇到点坑的。
  推荐购买vultr的$5/月的东京服务器。
## 远程连接
   ssh可以用win10自带的ubuntu，或者putty什么的。
   sftp推荐mobaxterm，可拖拽上传下载，简单好用，免费，绿色，不过没中文。
## 环境搭建
   系统在购买服务器的时候就选好了，自动安装。
   ssh连接服务器
   <pre><code>ssh root@服务器公网IP</code></pre>
   安装常用软件
   <pre><code>apt-get install vim wget curl git</code></pre>
### Node环境配置
  nvm安装
  <code>wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash</code>
  打开ssh新的窗口
  <code>nvm install v8.9.1</code>
  <code>nvm use 8.9.1</code>
  <code>nvm alias default 8.9.1</code>
  安装常用node包
  <code>npm i pm2 webpack vue-cli -g</code>
  以root用户身份在根目录下创建www目录,www目录下创建hello文件夹,里面就一个文件，hello.js,内容如下：
  <pre><code>
    const http = require('http')
    http.createServer(function(req,res) {
    res.writeHead(200,{'Content-Type':'text/plain'})
    res.end('hello world')
    }).listen(8081)
    console.log('server test')
  </code></pre>
  可以直接node命令跑起来，不过作为最简单的hello world是没啥问题的了，这里用pm2来跑，保证进程永远都活着,0秒的重载。
  pm2的几个常用命令：
  pm2 start preject 启动项目
  pm2 list 查看启动的应用
  pm2 show preject 查看详细信息
  pm2 logs 查看当前信息
  pm2 stop preject 停止preject
  pm2 delete preject 删除preject
  执行pm2 start hello你的服务就跑起来了，此时地址栏输入http://服务器IP:8081就会看到hello world
  执行pm2 list回看到详细信息
  阿里云专有网络的话需要设置下安全组规则
  在【云服务器】【安全组】里边点击【配置规则】 【添加安全组规则】 
  端口范围填写8081/8081（有域名的直接填写80/80，之后可以用nginx代理）
  授权对象填0.0.0.0/0
  其他默认
### Nginx安装
  <code> apt-get install nginx</code>
  nginx -v 查看版本号
  打开/etc/nginx/conf.d/文件夹，创建配置文件hello-8081.conf，内容如下：
   <pre><code>
    # 这里是代理端口号 hello和8081根据你的情况配置
      upstream nodejs__upstream {
          server 127.0.0.1:8081;
      }

      server {
          listen 80;
          # 域名配置
          server_name homi-tech.xyz www.homi-tech.xyz;
          location / {
              proxy_set_header Host  $http_host;
              proxy_set_header X-Real-IP  $remote_addr;  
              proxy_set_header X-Forwarded-For  $proxy_add_x_forwarded_for;
              proxy_set_header X-Nginx-proxy true;
              proxy_pass http://nodejs__upstream;
              proxy_redirect off;
          }
      }
  </code></pre>
  解释：配置文件类型必须以.conf结尾，文件名可自定义，为了方便记忆，遂以项目名加端口号的方式命名。
  完成配置后，执行sudo nginx -t查看是否配置成功
  service nginx start 启动服务
  sudo nginx -s reload 重启服务
  解析你的域名到你的服务器ip，这样就可以通过访问hello.example.com代理服务器的8081端口，Nginx在这里的作用就是让你可以在一台服务器跑多个Node项目。


    
