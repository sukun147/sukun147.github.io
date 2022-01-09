# 环境变量字符数解决方案


<!--more-->

各位可能遇见过这样一个：`Path` 变量字符数超过1023个，无法再继续添加了。

其实你可以另外新建一个环境变量（比如我的是`myEnvExtension`），然后在这个变量下添加你要添加的 `Path`，然后再在 `Path` 中添加一个变量值为 `%myEnvExtension%` 即可。

![image-20220107181734504](https://s2.loli.net/2022/01/07/BjuasOCpk2HUDzr.png)

![image-20220107181857756](https://s2.loli.net/2022/01/07/2EJYS7FLlqy8j3a.png)

