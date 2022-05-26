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

### 转化第一步：下载CTP文件，并重命名

从CTP官网，下载CTP接口文件，解压缩后拷贝`xxx_linux`文件下的所有文件，至某一文件夹下，文件清单：

```
thostmduserapi_se.so
thosttraderapi_se.so
ThostFtdcUserApiStruct.h
ThostFtdcUserApiDataType.h
ThostFtdcTraderApi.h
ThostFtdcMdApi.h
error.xml
error.dtd
```

重命名其中的.so文件：

```
thostmduserapi_se.so -> libthostmduserapi_se.so
thosttraderapi_se.so -> libthosttraderapi_se.so
```

这一步完成后，得到一个新文件夹，其下的文件清单：

```
libthostmduserapi_se.so
libthosttraderapi_se.so
ThostFtdcUserApiStruct.h
ThostFtdcUserApiDataType.h
ThostFtdcTraderApi.h
ThostFtdcMdApi.h
error.xml
error.dtd
```

### 转化第二步：创建.i文件，并生成中间文件

在上一步创建的文件夹下，创建两个`.i`文件：

```
thostmduserapi.i
thosttraderapi.i
```

#### thostmduserapi.i文件内容：

```
%module(directors="1") thostmduserapi
%{
#include "ThostFtdcMdApi.h"
#include "iconv.h"
%}

%feature("director") CThostFtdcMdSpi;
%ignore THOST_FTDC_VTC_BankBankToFuture;
%ignore THOST_FTDC_VTC_BankFutureToBank;
%ignore THOST_FTDC_VTC_FutureBankToFuture;
%ignore THOST_FTDC_VTC_FutureFutureToBank;
%ignore THOST_FTDC_FTC_BankLaunchBankToBroker;
%ignore THOST_FTDC_FTC_BrokerLaunchBankToBroker;
%ignore THOST_FTDC_FTC_BankLaunchBrokerToBank;
%ignore THOST_FTDC_FTC_BrokerLaunchBrokerToBank;

%typemap(out) char[ANY], char[] {
    if ($1) {
        iconv_t cd = iconv_open("utf-8", "gb2312");
        if (cd != reinterpret_cast<iconv_t>(-1)) {
            char buf[4096] = {};
            char **in = &$1;
            char *out = buf;
            size_t inlen = strlen($1), outlen = 4096;

            if (iconv(cd, in, &inlen, &out, &outlen) != static_cast<size_t>(-1))
			{
				size_t size = outlen;
				while (size && (buf[size - 1] == '\0')) --size;
				resultobj = SWIG_FromCharPtrAndSize(buf, size);
			}
            iconv_close(cd);
        }
    }
}

%typemap(in) char *[] {
  /* Check if is a list */
  if (PyList_Check($input)) {
    int size = PyList_Size($input);
    int i = 0;
    $1 = (char **) malloc((size+1)*sizeof(char *));
    for (i = 0; i < size; i++) {
      PyObject *o = PyList_GetItem($input, i);
      if (PyString_Check(o)) {
        $1[i] = PyString_AsString(PyList_GetItem($input, i));
      } else {
        free($1);
        PyErr_SetString(PyExc_TypeError, "list must contain strings");
        SWIG_fail;
      }
    }
    $1[i] = 0;
  } else {
    PyErr_SetString(PyExc_TypeError, "not a list");
    SWIG_fail;
  }
}

// This cleans up the char ** array we malloc'd before the function call
%typemap(freearg) char ** {
  free((char *) $1);
}
%include "ThostFtdcUserApiDataType.h"
%include "ThostFtdcUserApiStruct.h"
%include "ThostFtdcMdApi.h"
```

#### thosttraderapi.i文件内容：

```
%module(directors="1") thosttraderapi 
%{ 
#include "ThostFtdcTraderApi.h"
#include "iconv.h"
%}

%typemap(out) char[ANY], char[] {
    if ($1) {
        iconv_t cd = iconv_open("utf-8", "gb2312");
        if (cd != reinterpret_cast<iconv_t>(-1)) {
            char buf[4096] = {};
            char **in = &$1;
            char *out = buf;
            size_t inlen = strlen($1), outlen = 4096;

            if (iconv(cd, (char **)in, &inlen, &out, &outlen) != static_cast<size_t>(-1))
			{
				size_t size = outlen;
				while (size && (buf[size - 1] == '\0')) --size;
				resultobj = SWIG_FromCharPtrAndSize(buf, size);
			}
            iconv_close(cd);
        }
    }
}
%feature("director") CThostFtdcTraderSpi; 
%ignore THOST_FTDC_VTC_BankBankToFuture;
%ignore THOST_FTDC_VTC_BankFutureToBank;
%ignore THOST_FTDC_VTC_FutureBankToFuture;
%ignore THOST_FTDC_VTC_FutureFutureToBank;
%ignore THOST_FTDC_FTC_BankLaunchBankToBroker;
%ignore THOST_FTDC_FTC_BrokerLaunchBankToBroker;
%ignore THOST_FTDC_FTC_BankLaunchBrokerToBank;
%ignore THOST_FTDC_FTC_BrokerLaunchBrokerToBank;  
%feature("director") CThostFtdcTraderSpi; 
%include "ThostFtdcUserApiDataType.h"
%include "ThostFtdcUserApiStruct.h" 
%include "ThostFtdcTraderApi.h"
```

运行两次`swig`命令，生成针对两个`.i`的中间文件，命令如下：

```
swig -threads -py3 -c++ -python thostmduserapi.i
```

```
swig -threads -py3 -c++ -python thosttraderapi.i
```

### 转化第二步：创建make文件，完成转化

在上一步创建的文件夹下，创建两个`make`文件：

```
make_mduserapi
make_traderapi
```

#### make_mduserapi文件内容：

```
OBJS=thostmduserapi_wrap.o
INCLUDE=-I./ -I/usr/local/include/python3.7m

TARGET=_thostmduserapi.so
CPPFLAG=-shared -fPIC
CC=g++
LDLIB=-L. -lthostmduserapi_se
$(TARGET) : $(OBJS)
	$(CC) $(CPPFLAG) $(INCLUDE) -o $(TARGET) $(OBJS) $(LDLIB)
$(OBJS) : %.o : %.cxx
	$(CC) -c -fPIC $(INCLUDE) $< -o $@
clean:
	-rm -f $(OBJS)
	-rm -f $(TARGET)
```

#### make_mduserapi文件内容：

```
OBJS=thosttraderapi_wrap.o
INCLUDE=-I./ -I/usr/local/include/python3.7m

TARGET=_thosttraderapi.so
CPPFLAG=-shared -fPIC
CC=g++
LDLIB=-L. -lthosttraderapi_se
$(TARGET) : $(OBJS)
	$(CC) $(CPPFLAG) $(INCLUDE) -o $(TARGET) $(OBJS) $(LDLIB)
$(OBJS) : %.o : %.cxx
	$(CC) -c -fPIC $(INCLUDE) $< -o $@
clean:
	-rm -f $(OBJS)
	-rm -f $(TARGET)
```

运行两次`make`命令，生成两个`.so`文件，命令如下：

```
make -f make_mduserapi
```

```
make -f make_traderapi
```

最终，生成两个新的.so文件，查找该文件夹下，挑选出以下文件，即可投入使用：

```
_thostmduserapi.so
_thosttraderapi.so
libthostmduserapi_se.so
libthosttraderapi_se.so
thostmduserapi.py
thosttraderapi.py
YOUR_SOURCE_CODE.py
```

### FAQ



