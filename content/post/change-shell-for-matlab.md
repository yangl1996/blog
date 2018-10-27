---
date: "2018-10-27"
title: 更换MATLAB使用的Shell
---

最近需要使用MATLAB的coder工具，把一个MATLAB函数打包成C代码给别的项目用。不过，一开始总是报出奇怪的错，仔细一看以后发现`fish: Unknown command ' -r -p`，看来MATLAB coder实际上在用shell做一些操作，但是默认我的shell是bash，所以一些命令不兼容了。但是我也不想为了MATLAB就换回bash，所以稍微google了一下。事实上，MATLAB会依次检查`MATLAB_SHELL`和`SHELL`这两个env，因此只需要

```
launchctl setenv MATLAB_SHELL /bin/bash
```

然后重启MATLAB，问题就解决了。