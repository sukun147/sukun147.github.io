# 青龙面板教程


<!--more-->

# 青龙面板教程

## 前期准备

首先你要有一台服务器（国内鸡需要有代理），一个域名和 SSL证书（可选）。我这里的服务器系统为 Ubuntu-20.03

ssh 登录上去，安装 docker，此处采用官方脚本一键安装并设置开机自启。

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo systemctl enable docker
```

## 部署

### docker启动容器

启动容器（项目地址：https://github.com/whyour/qinglong）：

```bash
docker run -dit \
  -v $PWD/ql/config:/ql/config \
  -v $PWD/ql/log:/ql/log \
  -v $PWD/ql/db:/ql/db \
  -v $PWD/ql/repo:/ql/repo \
  -v $PWD/ql/raw:/ql/raw \
  -v $PWD/ql/scripts:/ql/scripts \
  -p 5700:5700 \
  --name qinglong \
  --hostname qinglong \
  --restart unless-stopped \
  whyour/qinglong:latest
```

其中`-dit`是后台运行、交互式操作、终端，`-v`是映射，`-p`是端口转发，`--name`是容器命名，`--hostname`是主机名，`--restart unless-stopped`是容器退出时总是重启，镜像是`whyour/qinglong:latest`

接着放行 5700 端口：

```bash
ufw allow 5700
```

在浏览器地址栏键入`ip:5700`即可访问，由于是第一次访问，需要进行初始化：

![image-20220218160423526](https://s2.loli.net/2022/02/18/hzGobqmvVrQP8Wc.png)

通知设置，可以采用钉钉、server酱等方式

![image-20220218160517709](https://s2.loli.net/2022/02/18/pmfWAN81VU4atbk.png)

然后是账号密码设置，本处不赘述。

### 反向代理

接着就可以正常使用了，但你肯定不会满足于用`ip:5700`的方式访问的，于是我还需要用到Nginx反向代理：

本处默认你已经部署好了 SSL证书，因此只讲解修改Nginx配置文件（可以用`nginx -t`指令找到配置文件位置）：

```nginx
server {
    listen 80;
    listen [::]:80;
    listen 81 http2;
    server_name 你的域名; #修改本处域名
    root /usr/share/nginx/html;
    location / {
        #反向代理
        proxy_ssl_server_name on;
        proxy_pass http://ip:5700; #修改本处ip
        proxy_set_header Accept-Encoding '';
        #过滤器模块
        sub_filter "ip:5700" "你的域名"; #修改本处
        sub_filter_once off;
    }
}
```

修改完后重启 Nginx 服务

```bash
nginx -s reload
```

这样一来你就可以在浏览器中键入域名来访问青龙面板了！

### 配置环境变量

在浏览器地址栏键入`m.jd.com`访问京东手机版网页，使用手机验证码登录，按下`F12`在Cookies中找到`pt_key、pt_pin`，复制下来，在面板中点击环境变量，点击添加变量，将刚刚复制的内容填入对应位置（一行一个 cookie）：

![image-20220218162323115](https://s2.loli.net/2022/02/18/KTD2sV1edASbl8o.png)

### 设置拉取脚本任务

在面板中的定时任务，点击添加任务：

![拉取脚本](https://s2.loli.net/2022/02/18/xFyOtdaQAb8mNXr.png)

意为每日 0时0分 执行命令栏中的指令（项目地址：https://github.com/Yun-City/City）：

```bash
ql repo https://github.com/Yun-City/City.git "jd_|jx_|gua_|jddj_|getJDCookie" "activity|backUp" "^jd[^_]|USER|function|utils|sendnotify|ZooFaker_Necklace|jd_Cookie|JDJRValidator_|sign_graphics_validate|ql|magic|cleancart_activity"
```

添加完成后运行一次该任务即可拉取脚本，然后就可以静静等待其运行了。

## 常见问题解决

### ql指令

```bash
# 更新并重启青龙
ql update                                                    
# 运行自定义脚本extra.sh
ql extra                                                     
# 添加单个脚本文件
ql raw <file_url>                                             
# 添加单个仓库的指定脚本
ql repo <repo_url> <whitelist> <blacklist> <dependence> <branch>   
# 删除旧日志
ql rmlog <days>                                              
# 启动tg-bot
ql bot                                                       
# 检测青龙环境并修复
ql check                                                     
# 重置登录错误次数
ql resetlet                                                  
# 禁用两步登录
ql resettfa                                                  

# 依次执行，如果设置了随机延迟，将随机延迟一定秒数
task <file_path>                                             
# 依次执行，无论是否设置了随机延迟，均立即运行，前台会输出日，同时记录在日志文件中
task <file_path> now                                         
# 并发执行，无论是否设置了随机延迟，均立即运行，前台不产生日，直接记录在日志文件中，且可指定账号执行
task <file_path> conc <env_name> <account_number>(可选的) 
# 指定账号执行，无论是否设置了随机延迟，均立即运行 
task <file_path> desi <env_name> <account_number>             
```

- file_url: 脚本地址
- repo_url: 仓库地址
- whitelist: 拉取仓库时的白名单，即就是需要拉取的脚本的路径包含的字符串
- blacklist: 拉取仓库时的黑名单，即就是需要拉取的脚本的路径不包含的字符串
- dependence: 拉取仓库需要的依赖文件，会直接从仓库拷贝到scripts下的仓库目录，不受黑名单影响
- branch: 拉取仓库的分支
- days: 需要保留的日志的天数
- file_path: 任务执行时的文件路径
- env_name: 任务执行时需要并发或者指定时的环境变量名称
- account_number: 任务执行时指定某个环境变量需要执行的账号序号

### 依赖

一键安装依赖：

```bash
docker exec -it qinglong bash -c "$(curl -fsSL https://ghproxy.com/https://raw.githubusercontent.com/Yun-City/City/main/Shell/QLOneKeyDependency.sh | sh)"
```

手动安装：

- npm install -g png-js
- npm install -g date-fns
- npm install -g axios
- npm install -g crypto-js
- npm install -g ts-md5
- npm install -g tslib
- npm install -g @types/node
- npm install -g requests

当然你也可以在面板的依赖管理处添加

![image-20220218162530574](https://s2.loli.net/2022/02/18/NPRT59v2HfmcwFJ.png)

### HTTP2问题

你可能会遇到这样的报错：

HTTP/2 stream 0 was not closed cleanly: PROTOCOL_ERROR (err 1)

只需要在容器中执行以下命令即可解决：

```bash
git config --global http.version HTTP/1.1
```


