# 本地定时脚本


<!--more-->

# 本地定时脚本

## python脚本

本脚本修改自[GitHub开源项目](https://github.com/fullstack-sake/legym_fk)

本脚本功能：乐健体育跑 3.5km 并活动打卡。

要执行本脚本，你需要安装`legym`模块

```bash
pip install -i https://test.pypi.org/simple legym==0.4
```

代码：

```python
from _datetime import datetime
import legym


# 注意修改本处账户密码
def run(activity, username='#', password='#', distance='3.5'):
    print("Login...", end="")
    user = legym.login(username, password)
    print("success")
    print("Running...", end="")
    actual_distance, success = user.running(float(distance))
    if success:
        print(f"{actual_distance} km")
    else:
        print("failed")
    print("Registering...", end="")
    _, success, _ = user.register(name=activity)
    if success:
        print("success")
    else:
        print("failed")
    print("Signing...", end="")
    results = user.sign()
    for result in results:
        if result[1]:
            print("success")
        else:
            print("failed")


def weekday():
    week = datetime.today().isoweekday()
    week_chinese = {1: '周一', 2: '周二', 3: '周三', 4: '周四', 5: '周五', 6: '周六', 7: '周日'}
    return week_chinese[week]


if __name__ == '__main__':
    act = '第三空间'+weekday()+'沙河校区体育场'
    run(act)

```

## 任务计划程序

打开计算机管理

![计算机管理](https://s2.loli.net/2022/02/22/sR2KPtvbkWrE4BV.png)

点击 系统工具 >>> 任务计划程序 >>> 创建基本任务

![任务计划程序](https://s2.loli.net/2022/02/22/eboSws17zxNYmXg.png)

自主填写任务名称与描述 >>> 选择每天 >>> 选择执行时间 >>> 选择启动程序

![触发器](https://s2.loli.net/2022/02/22/TRDrPoV49cHmqsX.png)

![操作](https://s2.loli.net/2022/02/22/JbP16zZmIl8LaSo.png)

![启动程序](https://s2.loli.net/2022/02/22/NYdfRmHoDpxq3lZ.png)

- 程序或脚本：python.exe 或 pythonw.exe 的路径，选择后者不会出现 IDE 窗口
- 添加参数：python脚本路径
- 起始于：Python编译器路径

完成后可在 任务计划程序库 中查看。

![任务计划程序库](https://s2.loli.net/2022/02/22/lIj9sSHC1uJZDRd.png)

