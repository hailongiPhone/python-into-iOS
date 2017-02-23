### 如何在iOS项目中运行 Python代码


##### Embed Python in to iOS
iOS中嵌入Python脚本简单概括来说包括两个部分，
  1. 主程序iOS App  
     主程序可以以C语言的函数调用方式，控制解释器调用运行Python代码，Python代码可以通过一定的规则调用C代码  

  2. 在主程序中可以运行的Python运行环境。  
  Python运行环境包括了解释器，标准库。解释器初始化的过程中会按照PythonHome指定的路径找到标准库并加载，从而完成Python运行环境的初始化过程。

##### 具体步骤  
  以Python3.5为例添加步骤如下：
  1. 添加解释器  
     Python解释器是以Framework的方式添加到iOS项目中。  
     [下载地址：Python-Apple-support](https://github.com/pybee/Python-Apple-support)  
     3.5版本包含了4个框架文件，依次添加
```
	     Python.framework  (示例代码中由于文件过大没有添加)
	     OpenSSl.framework
	     XZ.framework
	     BZip2.framework
```
  唯一需要注意的是要添加依赖库：
```
         libz.tbd
         sqlite3.tbd  
```
  2. 添加标准库
     python35.zip文件在Python.Framework文件夹中路径如下  
         Python.Framework/Resources/lib/py

     解释器初始化时搜索标准库有一定的路径格式，需要注意
     注意两点：
     1. 搜索路径由代码指定Py_SetPythonHome()指定
     2. home的路径格式
     └── pythonHome(可以任意名称)
              └── lib
              ├── python3.5
              └── python35.zip

  3. 代码中初始化解释器  
    * 指定PythonHome
    * 调用Py_Initialize 完成初始化
```
  NSString * python_home = [[[NSBundle mainBundle] bundlePath] stringByAppendingPathComponent:@"python"];
  wchar_t * wpython_home = Py_DecodeLocale([python_home   UTF8String], NULL);
  Py_SetPythonHome(wpython_home);
  Py_Initialize();
  PyEval_InitThreads(); //初始化多线程
  PyRun_SimpleString("print('---tata---')");
```
  4. 添加Module搜索路径
     可以通过sys添加搜索路径，可以通过这种方式指定加载沙盒中的代码，可以实现热更新。
```
  NSString * path = @"modulePath";
  PyObject *sys = PyImport_ImportModule("sys");
  PyObject *pathPO = PyObject_GetAttrString(sys, "path");
  PyList_Append(pathPO, PyUnicode_FromFormat([path UTF8String]));
  PyList_Append(pathPO, PyUnicode_FromFormat("."));
  Py_XDECREF(pathPO);
  Py_XDECREF(sys);
```
