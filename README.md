# 获得Python版本的CTP接口 - Linux版本

本Repo的方法，总结自 https://github.com/nicai0609/Python-CTPAPI

### 缘由和目标

* 上期技术CTP系统提供的API是C++版本；
* C++开发的普及程度较低，而Python有更广大的使用人群和开箱即用的库；
* Swig.org 提供将C++版本的API转化为Python版本API的能力；
* 目标1：当CTP版本更新时，能够自助完成转化，确保稳定和安全；
* 目标2：将C++版本的API转化为Python版本，并确保函数方法名一致；
* 目标3：本Repo仅提供Linux版本，确保知识维护简单清晰。

### 免责声明

使用本Repo提供的方法，说明你同意：
* 本Repo提供的方法，不代表任何组织和个人；
* 是否采用本Repo提供的方法，完全基于你自己的判断，若产生任何与之相关的损失，本人一概不负任何责任。

### 以终为始：投产文件清单

转化成功后，投入生产使用的API相关的文件清单如下：

```
_thostmduserapi.so
_thosttraderapi.so
libthostmduserapi_se.so
libthosttraderapi_se.so
thostmduserapi.py
thosttraderapi.py
YOUR_SOURCE_CODE.py
```

业务代码文件（例如上述的`YOUR_SOURCE_CODE.py`），放在与API文件相同目录下，并确保将该目录赋值给环境变量`LD_LIBRARY_PATH`

```
export LD_LIBRARY_PATH=/path/to/the/ctp/api/files
```

业务代码可参考Demo：https://github.com/nicai0609/Python-CTPAPI/tree/master/demo



