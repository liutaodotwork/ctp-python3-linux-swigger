# 在Linux下转化Python版本的CTP接口

### 缘由和目标

* 上期技术CTP系统提供的API是C++版本
* C++开发的普及程度较低，而Python有更广大的使用人群和更为开箱即用的库
* Swig.org 提供将C++版本的API转化为Python版本API的能力
* 目标1：将C++版本的API转化为Python版本，并确保函数方法名一致
* 目标2：当CTP版本更新时，能够自助完成转化，确保稳定安全
* 目标3：本Repo仅提供Linux版本，确保知识维护简单清晰
