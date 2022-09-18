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

## 青龙面板部署

### docker启动容器

启动容器（项目地址：https://github.com/whyour/qinglong）：

```bash
docker run -dit \
  -v $PWD/ql:/ql/data \
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

### 设置脚本订阅

在订阅管理处添加订阅即可

![image-20220918091622807](https://image.sukun.xyz/img/image-20220918091622807.png)

```bash
ql repo https://ghproxy.com/https://github.com/KingRan/KR.git "jd_|jx_|jdCookie" "activity|backUp" "^jd[^_]|USER|utils|function|sign|sendNotify|ql|JDJR"
```

注意，本处使用了`https://ghproxy.com`代理，国外鸡可直接删除代理。

## 青龙面板实现每日签到

#### 安装依赖

首先进入容器

```bash
docker exec -it qinglong bash
```

```shell
apk add --no-cache gcc g++ python python-dev py-pip mysql-dev linux-headers libffi-dev openssl-dev
```

如果上述命令安装依赖失败，请使用下方命令

```bash
apk add --no-cache --virtual .build-deps gcc musl-dev python2-dev python3-dev libffi libffi-dev openssl openssl-dev
pip3 install pip setuptools --upgrade
pip3 install cryptography~=3.2.1
```

现在安装`dailycheckin`

```bash
pip3 install dailycheckin --upgrade
```

#### `/ql/scripts/config.json` 配置文件

详解参考[配置 - DailyCheckIn (sitoi.github.io)](https://sitoi.github.io/dailycheckin/settings/)

```json
{
  "DINGTALK_SECRET": "",
  "DINGTALK_ACCESS_TOKEN": "",
  "FSKEY": "",
  "SCKEY": "",
  "SENDKEY": "",
  "BARK_URL": "",
  "QMSG_KEY": "",
  "QMSG_TYPE": "",
  "TG_BOT_TOKEN": "",
  "TG_USER_ID": "",
  "TG_API_HOST": "",
  "TG_PROXY": "",
  "COOLPUSHSKEY": "",
  "COOLPUSHQQ": true,
  "COOLPUSHWX": true,
  "COOLPUSHEMAIL": true,
  "QYWX_KEY": "",
  "QYWX_CORPID": "",
  "QYWX_AGENTID": "",
  "QYWX_CORPSECRET": "",
  "QYWX_TOUSER": "",
  "QYWX_MEDIA_ID": "",
  "PUSHPLUS_TOKEN": "",
  "PUSHPLUS_TOPIC": "",
  "MERGE_PUSH": "",
  "IQIYI": [
    {
      "cookie": "__dfp=xxxxxx; QP0013=xxxxxx; QP0022=xxxxxx; QYABEX=xxxxxx; P00001=xxxxxx; P00002=xxxxxx; P00003=xxxxxx; P00007=xxxxxx; QC163=xxxxxx; QC175=xxxxxx; QC179=xxxxxx; QC170=xxxxxx; P00010=xxxxxx; P00PRU=xxxxxx; P01010=xxxxxx; QC173=xxxxxx; QC180=xxxxxx; P00004=xxxxxx; QP0030=xxxxxx; QC006=xxxxxx; QC007=xxxxxx; QC008=xxxxxx; QC010=xxxxxx; nu=xxxxxx; __uuid=xxxxxx; QC005=xxxxxx;"
    },
    {
      "cookie": "多账号 cookie 填写，请参考上面，cookie 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）"
    }
  ],
  "VQQ": [
    {
      "auth_refresh": "https://access.video.qq.com/user/auth_refresh?vappid=xxxxxx&vsecret=xxxxxx&type=qq&g_tk=&g_vstk=xxxxxx&g_actk=xxxxxx&callback=xxxxxx&_=xxxxxx",
      "cookie": "pgv_pvid=xxxxxx; pac_uid=xxxxxx; RK=xxxxxx; ptcz=xxxxxx; tvfe_boss_uuid=xxxxxx; video_guid=xxxxxx; video_platform=xxxxxx; pgv_info=xxxxxx; main_login=xxxxxx; vqq_access_token=xxxxxx; vqq_appid=xxxxxx; vqq_openid=xxxxxx; vqq_vuserid=xxxxxx; vqq_refresh_token=xxxxxx; login_time_init=xxxxxx; uid=xxxxxx; vqq_vusession=xxxxxx; vqq_next_refresh_time=xxxxxx; vqq_login_time_init=xxxxxx; login_time_last=xxxxxx;"
    },
    {
      "auth_refresh": "多账号 refresh url，请参考上面，以实际获取为准",
      "cookie": "多账号 cookie 填写，请参考上面，cookie 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）"
    }
  ],
  "YOUDAO": [
    {
      "cookie": "JSESSIONID=xxxxxx; __yadk_uid=xxxxxx; OUTFOX_SEARCH_USER_ID_NCOO=xxxxxx; YNOTE_SESS=xxxxxx; YNOTE_PERS=xxxxxx; YNOTE_LOGIN=xxxxxx; YNOTE_CSTK=xxxxxx; _ga=xxxxxx; _gid=xxxxxx; _gat=xxxxxx; PUBLIC_SHARE_18a9dde3de846b6a69e24431764270c4=xxxxxx;"
    },
    {
      "cookie": "多账号 cookie 填写，请参考上面，cookie 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）"
    }
  ],
  "KGQQ": [
    {
      "cookie": "muid=xxxxxx; uid=xxxxxx; userlevel=xxxxxx; openid=xxxxxx; openkey=xxxxxx; opentype=xxxxxx; qrsig=xxxxxx; pgv_pvid=xxxxxx;"
    },
    {
      "cookie": "多账号 cookie 填写，请参考上面，cookie 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）"
    }
  ],
  "ONEPLUSBBS": [
    {
      "cookie": "acw_tc=xxxxxx; qKc3_0e8d_saltkey=xxxxxx; qKc3_0e8d_lastvisit=xxxxxx; bbs_avatar=xxxxxx; qKc3_0e8d_sendmail=xxxxxx; opcid=xxxxxx; opcct=xxxxxx; oppt=xxxxxx; opsid=xxxxxx; opsct=xxxxxx; opbct=xxxxxx; UM_distinctid=xxxxxx; CNZZDATA1277373783=xxxxxx; www_clear=xxxxxx; ONEPLUSID=xxxxxx; qKc3_0e8d_sid=xxxxxx; bbs_uid=xxxxxx; bbs_uname=xxxxxx; bbs_grouptitle=xxxxxx; opuserid=xxxxxx; bbs_sign=xxxxxx; bbs_formhash=xxxxxx; qKc3_0e8d_ulastactivity=xxxxxx; opsertime=xxxxxx; qKc3_0e8d_lastact=xxxxxx; qKc3_0e8d_checkpm=xxxxxx; qKc3_0e8d_noticeTitle=xxxxxx; optime_browser=xxxxxx; opnt=xxxxxx; opstep=xxxxxx; opstep_event=xxxxxx; fp=xxxxxx;"
    },
    {
      "cookie": "多账号 cookie 填写，请参考上面，cookie 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）"
    }
  ],
  "BAIDU": [
    {
      "data_url": "https://cdn.jsdelivr.net/gh/Sitoi/Sitoi.github.io/baidu_urls.txt",
      "submit_url": "http://data.zz.baidu.com/urls?site=https://sitoi.cn&token=xxxxxx",
      "times": 10
    },
    {
      "data_url": "多账号 data_url 链接地址，以实际获取为准",
      "submit_url": "多账号 submit_url 链接地址，以实际获取为准",
      "times": 10
    }
  ],
  "FMAPP": [
    {
      "blackbox": "eyJlcnJxxxxxx",
      "cookie": "sensorsdata2015jssdkcross=xxxxxx",
      "device_id": "xxxxxx-xxxx-xxxx-xxxx-xxxxxx",
      "fmversion": "xxxxxx",
      "os": "xxxxxx",
      "token": "xxxxxx.xxxxxx-xxxxxx-xxxxxx.xxxxxx-xxxxxx",
      "useragent": "xxxxxx"
    },
    {
      "blackbox": "多账号 blackbox 填写，请参考上面，blackbox 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）",
      "cookie": "多账号 cookie 填写，请参考上面，cookie 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）",
      "device_id": "多账号 device_id 填写，请参考上面，以实际获取为准",
      "fmversion": "多账号 fmVersion 填写，请参考上面，以实际获取为准",
      "os": "多账号 os 填写，请参考上面，以实际获取为准",
      "token": "多账号 token 填写，请参考上面，以实际获取为准",
      "useragent": "多账号 User-Agent 填写，请参考上面，以实际获取为准"
    }
  ],
  "TIEBA": [
    {
      "cookie": "BIDUPSID=xxxxxx; PSTM=xxxxxx; BAIDUID=xxxxxx; BAIDUID_BFESS=xxxxxx; delPer=xxxxxx; PSINO=xxxxxx; H_PS_PSSID=xxxxxx; BA_HECTOR=xxxxxx; BDORZ=xxxxxx; TIEBA_USERTYPE=xxxxxx; st_key_id=xxxxxx; BDUSS=xxxxxx; BDUSS_BFESS=xxxxxx; STOKEN=xxxxxx; TIEBAUID=xxxxxx; ab_sr=xxxxxx; st_data=xxxxxx; st_sign=xxxxxx;"
    },
    {
      "cookie": "多账号 cookie 填写，请参考上面，cookie 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）"
    }
  ],
  "BILIBILI": [
    {
      "cookie": "_uuid=xxxxxx; rpdid=xxxxxx; LIVE_BUVID=xxxxxx; PVID=xxxxxx; blackside_state=xxxxxx; CURRENT_FNVAL=xxxxxx; buvid3=xxxxxx; fingerprint3=xxxxxx; fingerprint=xxxxxx; buivd_fp=xxxxxx; buvid_fp_plain=xxxxxx; DedeUserID=xxxxxx; DedeUserID__ckMd5=xxxxxx; SESSDATA=xxxxxx; bili_jct=xxxxxx; bsource=xxxxxx; finger=xxxxxx; fingerprint_s=xxxxxx;",
      "coin_num": 0,
      "coin_type": 1,
      "silver2coin": true
    },
    {
      "cookie": "多账号 cookie 填写，请参考上面，cookie 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）",
      "coin_num": 0,
      "coin_type": 1,
      "silver2coin": true
    }
  ],
  "V2EX": [
    {
      "cookie": "_ga=xxxxxx; __cfduid=xxxxxx; PB3_SESSION=xxxxxx; A2=xxxxxx; V2EXSETTINGS=xxxxxx; V2EX_REFERRER=xxxxxx; V2EX_LANG=xxxxxx; _gid=xxxxxx; V2EX_TAB=xxxxxx;",
      "proxy": "使用代理的信息，无密码例子: http://127.0.0.1:1080 有密码例子: http://username:password@127.0.0.1:1080"
    },
    {
      "cookie": "多账号 cookie 填写，请参考上面，cookie 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）",
      "proxy": "使用代理的信息，无密码例子: http://127.0.0.1:1080 有密码例子: http://username:password@127.0.0.1:1080"
    }
  ],
  "WWW2NZZ": [
    {
      "cookie": "YPx9_2132_saltkey=xxxxxx; YPx9_2132_lastvisit=xxxxxx; YPx9_2132_sendmail=xxxxxx; YPx9_2132_con_request_uri=xxxxxx; YPx9_2132_sid=xxxxxx; YPx9_2132_client_created=xxxxxx; YPx9_2132_client_token=xxxxxx; YPx9_2132_ulastactivity=xxxxxx; YPx9_2132_auth=xxxxxx; YPx9_2132_connect_login=xxxxxx; YPx9_2132_connect_is_bind=xxxxxx; YPx9_2132_connect_uin=xxxxxx; YPx9_2132_stats_qc_login=xxxxxx; YPx9_2132_checkpm=xxxxxx; YPx9_2132_noticeTitle=xxxxxx; YPx9_2132_nofavfid=xxxxxx; YPx9_2132_lastact=xxxxxx;"
    },
    {
      "cookie": "多账号 cookie 填写，请参考上面，cookie 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）"
    }
  ],
  "SMZDM": [
    {
      "cookie": "__jsluid_s=xxxxxx; __ckguid=xxxxxx; device_id=xxxxxx; homepage_sug=xxxxxx; r_sort_type=xxxxxx; _zdmA.vid=xxxxxx; sajssdk_2015_cross_new_user=xxxxxx; sensorsdata2015jssdkcross=xxxxxx; footer_floating_layer=xxxxxx; ad_date=xxxxxx; ad_json_feed=xxxxxx; zdm_qd=xxxxxx; sess=xxxxxx; user=xxxxxx; _zdmA.uid=xxxxxx; smzdm_id=xxxxxx; userId=xxxxxx; bannerCounter=xxxxxx; _zdmA.time=xxxxxx;"
    },
    {
      "cookie": "多账号 cookie 填写，请参考上面，cookie 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）"
    }
  ],
  "MIMOTION": [
    {
      "max_step": "20000",
      "min_step": "10000",
      "password": "Sitoi",
      "phone": "18888xxxxxx"
    },
    {
      "max_step": "多账号 最大步数填写，请参考上面",
      "min_step": "多账号 最小步数填写，请参考上面",
      "password": "多账号 密码填写，请参考上面",
      "phone": "多账号 手机号填写，请参考上面"
    }
  ],
  "ACFUN": [
    {
      "password": "Sitoi",
      "phone": "18888xxxxxx"
    },
    {
      "password": "多账号 密码填写，请参考上面",
      "phone": "多账号 手机号填写，请参考上面"
    }
  ],
  "CLOUD189": [
    {
      "password": "Sitoi",
      "phone": "18888xxxxxx"
    },
    {
      "password": "多账号 密码填写，请参考上面",
      "phone": "多账号 手机号填写，请参考上面"
    }
  ],
  "MGTV": [
    {
      "params": "uuid=xxxxxx&uid=xxxxxx&ticket=xxxxxx&token=xxxxxx&device=iPhone&did=xxxxxx&deviceId=xxxxxx&appVersion=6.8.2&osType=ios&platform=iphone&abroad=0&aid=xxxxxx&nonce=xxxxxx&timestamp=xxxxxx&appid=xxxxxx&type=1&sign=xxxxxx&callback=xxxxxx"
    },
    {
      "params": "多账号 请求参数填写，请参考上面"
    }
  ],
  "PICACOMIC": [
    {
      "email": "Sitoi",
      "password": "xxxxxx"
    },
    {
      "email": "多账号 账号填写，请参考上面",
      "password": "多账号 密码填写，请参考上面"
    }
  ],
  "MEIZU": [
    {
      "draw_count": "1",
      "cookie": "aliyungf_tc=xxxxxx; logined_uid=xxxxxx; acw_tc=xxxxxx; LT=xxxxxx; MZBBS_2132_saltkey=xxxxxx; MZBBS_2132_lastvisit=xxxxxx; MZBBSUC_2132_auth=xxxxxx; MZBBSUC_2132_loginmember=xxxxxx; MZBBSUC_2132_ticket=xxxxxx; MZBBS_2132_sid=xxxxxx; MZBBS_2132_ulastactivity=xxxxxx; MZBBS_2132_auth=xxxxxx; MZBBS_2132_loginmember=xxxxxx; MZBBS_2132_lastcheckfeed=xxxxxx; MZBBS_2132_checkfollow=xxxxxx; MZBBS_2132_lastact=xxxxxx;"
    },
    {
      "draw_count": "多账号 抽奖次数设置",
      "cookie": "多账号 cookie 填写，请参考上面，cookie 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）"
    }
  ],
  "ZHIYOO": [
    {
      "cookie": "ikdQ_9242_saltkey=xxxxxx; ikdQ_9242_lastvisit=xxxxxx; ikdQ_9242_onlineusernum=xxxxxx; ikdQ_9242_sendmail=1; ikdQ_9242_seccode=xxxxxx; ikdQ_9242_ulastactivity=xxxxxx; ikdQ_9242_auth=xxxxxx; ikdQ_9242_connect_is_bind=xxxxxx; ikdQ_9242_nofavfid=xxxxxx; ikdQ_9242_checkpm=xxxxxx; ikdQ_9242_noticeTitle=1; ikdQ_9242_sid=xxxxxx; ikdQ_9242_lip=xxxxxx; ikdQ_9242_lastact=xxxxxx"
    },
    {
      "cookie": "多账号 cookie 填写，请参考上面，cookie 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）"
    }
  ],
  "WEIBO": [
    {
      "url": "https://api.weibo.cn/2/users/show?wm=xxxxxx&launchid=xxxxxx&b=xxxxxx&from=xxxxxx&c=xxxxxx&networktype=xxxxxx&v_p=xxxxxx&skin=xxxxxx&v_f=xxxxxx&lang=xxxxxx&sflag=xxxxxx&ua=xxxxxx&ft=xxxxxx&aid=xxxxxx&has_extend=xxxxxx&uid=xxxxxx&gsid=xxxxxx&sourcetype=&get_teenager=xxxxxx&s=xxxxxx&has_profile=xxxxxx"
    },
    {
      "url": "多账号 show_url 填写，请参考上面，show_url 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）"
    }
  ],
  "DUOKAN": [
    {
      "cookie": "user_id=xxxxxx; token=xxxxxx; user_gender=xxxxxx; device_id=xxxxxx; app_id=xxxxxx; build=xxxxxx; short_version=xxxxxx"
    },
    {
      "cookie": "多账号 cookie 填写，请参考上面，cookie 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）"
    }
  ],
  "CSDN": [
    {
      "cookie": "uuid_tt_dd=xxxxxx; _ga=xxxxxx; UserName=xxxxxx; UserInfo=xxxxxx; UserToken=xxxxxx; UserNick=xxxxxx; AU=768; UN=xxxxxx; BT=xxxxxx; p_uid=xxxxxx; Hm_up_6bcd52f51e9b3dce32bec4a3997715ac=xxxxxx; Hm_ct_6bcd52f51e9b3dce32bec4a3997715ac=xxxxxx; Hm_lvt_6bcd52f51e9b3dce32bec4a3997715ac=xxxxxx dc_sid=xxxxxx; c_segment=xxxxxx; dc_session_id=xxxxxx; csrfToken=xxxxxx; c_first_ref=xxxxxx; c_first_page=xxxxxx; c_page_id=xxxxxx; announcement-new=xxxxxx; log_Id_click=xxxxxx; c_pref=xxxxxx; c_ref=xxxxxx; dc_tos=xxxxxx; log_Id_pv=xxxxxx; log_Id_view=xxxxxx"
    },
    {
      "cookie": "多账号 cookie 填写，请参考上面，cookie 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）"
    }
  ],
  "WZYD": [
    {
      "data": "areaId=xxxxxx&roleId=xxxxxx&gameId=xxxxxx&serverId=xxxxxx&gameOpenid=xxxxxx&userId=xxxxxx&appVersion=xxxxxx&cClientVersionName=xxxxxx&platid=xxxxxx&source=xxxxxx&algorithm=xxxxxx&version=xxxxxx&timestamp=xxxxxx&appid=xxxxxx&openid=xxxxxx&sig=xxxxxx&encode=2&msdkEncodeParam=xxxxxx&cSystem=xxxxxx&h5Get=xxxxxx&msdkToken=&appOpenid=xxxxxx"
    },
    {
      "data": "多账号 data 填写，请参考上面，data 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）"
    }
  ],
  "WOMAIL": [
    {
      "url": "https://nyan.mail.wo.cn/cn/sign/index/index?mobile=xxxxxx&userName=&openId=xxxxxx",
      "pause21days": true,
      "password": "Sitoi",
      "phone": "18888xxxxxx"
    },
    {
      "url": "多账号 url 填写，请参考上面，url 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）",
      "pause21days": true,
      "password": "多账号 密码填写，请参考上面",
      "phone": "多账号 手机号填写，请参考上面"
    }
  ],
  "HEYTAP": [
    {
      "cookie": "sa_distinct_id=xxxxxx;Personalized=xxxxxx;s_channel=xxxxxx;source_type=xxxxxx;app_param=xxxxxx;ENCODE_TOKENSID=xxxxxx;scene_id=xxxxxx;apkPkg=xxxxxx;exp_id=;app_utm=xxxxxx;TOKENSID=xxxxxx;strategy_id=xxxxxx;referer=;experiment_id=xxxxxx;section_id=;s_version=xxxxxx;app_innerutm=xxxxxx;retrieve_id=;log_id=;",
      "useragent": "xxxxxx",
      "draw": false
    },
    {
      "cookie": "多账号 cookie 填写，请参考上面，cookie 以实际获取为准（遇到特殊字符如双引号\" 请加反斜杠转义）",
      "useragent": "多账号 User-Agent 填写，请参考上面，以实际获取为准",
      "draw": false
    }
  ],
  "UNICOM": [
    {
      "mobile": "18888xxxxxx",
      "password": "xxxxxx",
      "app_id": "xxxxxx"
    },
    {
      "mobile": "多账号 手机号",
      "password": "多账号 密码",
      "app_id": "多账号 appId"
    }
  ],
  "EVERPHOTO": [
    {
      "mobile": "+8618888xxxxxx",
      "password": "xxxxxx"
    },
    {
      "mobile": "多账号 手机号",
      "password": "多账号 密码"
    }
  ],
  "SSPANEL": [
    {
      "email": "邮箱",
      "password": "密码",
      "url": "https://sitoi.cn"
    },
    {
      "email": "多账号 邮箱",
      "password": "多账号 密码",
      "url": "https://sitoi.cn"
    }
  ]
}
```

#### 配置定时任务

运行全部脚本：

![image-20220418143016914](https://s2.loli.net/2022/04/18/b1p6cGCVoNQ4zXY.png)

定时更新签到脚本：

![image-20220418143118817](https://s2.loli.net/2022/04/18/ujzXHKWGfJ1xN4Z.png)

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
ql repo <repo_url> <whitelist> <blacklist> <dependence> <branch> <extensions>
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

### HTTP2问题

你可能会遇到这样的报错：

HTTP/2 stream 0 was not closed cleanly: PROTOCOL_ERROR (err 1)

只需要在容器中执行以下命令即可解决：

```bash
git config --global http.version HTTP/1.1
```


