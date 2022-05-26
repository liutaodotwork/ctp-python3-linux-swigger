# 获得Python版本的CTP接口 - Linux版本

本Repo的方法，总结自 https://github.com/nicai0609/Python-CTPAPI

### 缘由和目标

* 上期技术CTP系统提供的API是C++版本
* C++开发的普及程度较低，而Python有更广大的使用人群和开箱即用的库
* Swig.org 提供将C++版本的API转化为Python版本API的能力
* 目标1：当CTP版本更新时，能够自助完成转化，确保稳定和安全
* 目标2：将C++版本的API转化为Python版本，并确保函数方法名一致
* 目标3：本Repo仅提供Linux版本，确保知识维护简单清晰

### 免责声明

使用本Repo提供的方法，说明你同意：
* 本Repo提供的方法，不代表任何组织和个人
* 是否采用本Repo提供的方法，完全基于你自己的判断，若产生任何与之相关的损失，本人一概不负任何责任

### 最终目标

转化成功后，投产时使用的文件清单如下：

···
_thostmduserapi.so
_thosttraderapi.so
libthostmduserapi_se.so
libthosttraderapi_se.so
thostmduserapi.py
thosttraderapi.py
···
