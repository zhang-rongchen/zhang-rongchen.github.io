---
layout: post
title:  C++调用python
date:   2020-07-19 11:49:45 +0200
categories: C++
---
为了方便环境管理使用Anaconda创建的Python环境

### 1. CMakeList

​	代码中需要导入<python.h>头文件，如果要涉及到传图像参数还需要导入<numpy.h>，因此需要将对应的文件路径写到CMakeList文件中。

```CMake
cmake_minimum_required(VERSION 3.12)
project(learn1_1)
set(CMAKE_CXX_STANDARD 14)
set(DCMAKE_PREFIX_PATH "/home/zrc/anaconda3/envs/tf14")
set(PYTHON_INCLUDE_DIRS "/home/zrc/anaconda3/envs/tf14/include/python3.6m/")
set(NUMPY_INCLUDE_DIRS "/home/zrc/anaconda3/envs/tf14/lib/python3.6/site-packages/numpy/core/include/")
set(CMAKE_CURRENT_SOURCE_DIR "/home/zrc/CLionProjects/learn1_1")
INCLUDE_DIRECTORIES(${PYTHON_INCLUDE_DIRS} ${OpenCV_INCLUDE_DIRS} ${NUMPY_INCLUDE_DIRS} ${CMAKE_CURRENT_SOURCE_DIR}/include)
link_directories(/home/zrc/anaconda3/envs/tf14/lib/python3.6/config-3.6m-x86_64-linux-gnu)
set(PYTHON_LIBRARIES "/home/zrc/anaconda3/envs/tf14/lib/libpython3.6m.so")
add_executable(learn1_1 main.cpp include/retrieval.h retrieval.cpp)
target_link_libraries(learn1_1 ${PYTHON_LIBRARIES}
-I/usr/local/include/opencv -I/usr/local/include -L/usr/local/lib
-lopencv_dnn -lopencv_highgui -lopencv_ml
-lopencv_objdetect -lopencv_shape -lopencv_stitching
-lopencv_superres -lopencv_videostab -lopencv_calib3d
-lopencv_videoio -lopencv_imgcodecs -lopencv_features2d
-lopencv_video -lopencv_photo -lopencv_imgproc -lopencv_flann
-lopencv_viz -lopencv_core
)
```

### 2. C++调用Python相关函数说明

```c++
void Py_Initialize()
```

​	该函数的主要功能是：初始化 Python 运行环境，该函数必须在调用其它 Python 函数前首先调用，要注意该函数是无参数、无返回值。



```c++
bool Py_IsInitialized();
```

​	该函数的主要功能是：检查初始化 Python 运行环境是否成功，成功返回 true，否则返回 false。



```c++
PyAPI_FUNC(int) PyRun_SimpleStringFlags(const char *, PyCompilerFlags *);
```

​	该函数的主要功能是：执行一段 Python 语句，例如 import sys 等简单语句，一般用于加载 Python 解释器相关路径。



```c++
PyObject *PyImport_ImportModule(const char *szModuleName);
```

​	函数作用： 加载 Python 模块，参数为 Python 的模块名（文件名），返回加载模块对象的指针。 注意： 模块名不能为 test，否则不能得到想要的结果，因为 Python 内部对 test 有特殊定义。



```c++
PyObject *PyModule_GetDict(PyObject *pModule);
```

​	函数作用： 获取 Python 模块字典，参数为 Python 的类名或者函数名等，返回获取字典对象的指针。



```c++
PyObject *PyDict_GetItemString(PyObject *pDict, const char *key);
```

​	从字典模块中获取指定的对象，其中：

- *pDict 为将要查找的字典模块；
- const char *key 为要查找 Python 模块中的函数或者类名，注意此处是 const char 类型；



```c++
// python3
PyObject *PyInstanceMethod_New(PyObject *pClass);
// python2
PyObject *PyInstance_New(PyObject *pClass);
```

​	该函数作用是实例化 Python 类，生成类对象，其中 *pClass 为 Python 的类名。



```c++
PyObject *PyObject_CallMethod(PyObject *pInstance, const char *pMethodName, const  char *pFormat, ...);
```

​	函数作用： 获取 Python 模块字典，参数为 Python 的类名或者函数名等，返回获取字典对象的指针。



```c++
PyObject *PyModule_GetDict(PyObject *pModule);
```

​	该函数主要功能是调用类方法，其中：

- *pInstance：是由 PyInstanceMethod_New 函数返回的类的实例
- *pMethodName：是调用类的类名
- *pFormat：传递给类方法的格式化类型字符串
- *...：给类方法传递的可变参数



```c++
PyObject *PyObject_CallFunction(PyObject *pFunction, const char *pFormat, ...);
```

​	该函数主要功能是调用模块中的函数，其中：

- *pFunction：调用的函数名指针
- *pFormat：传递给模块函数的参数类型格式化字符串
- *...：传给模块函数的参数列表



```c++
PyArg_Parse(PyObject *pArgs, const char *pFormat, ...);
```

​	该函数用于解析 Python 函数或者方法的返回值，其中：

- *pArgs：为 Python 的返回值
- *pFormat：为返回值类型的格式化字符串
- *...：返回值的数据指针



```c++
Py_DECREF(pObject);
```

​	该函数用于释放之前创建的模块、函数、类、实例方法等。



```c++
Py_Finalize();
```

​	该函数用于释放之前创建的 Python 运行环境。

### 3. C++调用Python代码

#### 3.1 简单执行python代码

```c++
#include <iostream>
#include <Python.h>

int main(int argc, char *argv[])
{
    Py_Initialize();

    PyRun_SimpleString("import sys");
    PyRun_SimpleString("print('hello world')");

    Py_Finalize();
    return 0;
}
```

​	

#### 3.2 调用Python函数

##### 3.2.1 无参数调用

```c++
#include <iostream>
#include <Python.h>

int main(int argc, char *argv[])
{
    // 初始化 Python 解释器环境
    Py_Initialize();

     // 执行 Python 语句，设置本地根目录
    PyRun_SimpleString("import sys");
    PyRun_SimpleString("sys.path.append('./')");
    
    // 声明python对象
    PyObject *pModule = NULL;
    PyObject *pDict = NULL;
    PyObject *pFunc = NULL;

    // 导入需要执行的python脚本
    pModule = PyImport_ImportModule("test");
    
    // 获取导入的python脚本内所有子模块名称
    pDict = PyModule_GetDict(pModule);
    
    // 获取导入的python脚本内子模块中名称为helloworld的函数
    pFunc = PyDict_GetItemString(pDict, "helloworld");
    
    // 执行该函数
    PyEval_CallObject(pFunc, NULL);
    // PyObject_CallObject(pFunc, NULL); 与上句代码等价
    Py_Finalize();
    return 0;
}
```

​	

##### 3.2.2 有参数调用

​	python代码如下：

```c++
def get_name_age(name, age):
    print("name is {}, age is {}".format(name, age))
    return name
```

​	C++ 代码中修改相对应的函数调用和传参代码，相应的部分代码如下：

```c++
int main(int argc, char *argv[])
{
    ....
    pFunc = PyDict_GetItemString(pDict, "get_name_age");

    pArgs = PyTuple_New(2);
    PyTuple_SetItem(pArgs, 0, Py_BuildValue("s", "Jack"));
    PyTuple_SetItem(pArgs, 1, Py_BuildValue("i", 18));

    pRet = PyObject_CallObject(pFunc, pArgs);
    if (pRet != NULL)
    {
        char *ret_str = NULL;
        PyArg_Parse(pRet, "s", &ret_str);
        std::cout << "python return value is : " << ret_str << std::endl;
    }

    ...
    return 0;
}
```

###### 	传参

```c++
pArgs = PyTuple_New(2);
```

​	创建传参元组，在 C++ 中调用 Python 代码传参时，需要将参数组装成元组来传递，这里的 PyTuple_New(2) 表示要传递 2 个参数，可以根据实际情况，以此类推。例如下面两行代码表示要传递的参数，其中 s 代表要传递的参数为字符串，i 代表要传递的参数为整型。

```c++
 PyTuple_SetItem(pArgs, 0, Py_BuildValue("s", "Jack"));
 PyTuple_SetItem(pArgs, 1, Py_BuildValue("i", 18));
 // 0, 1 代表传参的序号，假设有 5 个参数，则分别用 0，1，2, 3，4 来区别
```

###### 	函数返回值

```c++
    pRet = PyObject_CallObject(pFunc, pArgs);
    if (pRet != NULL)
    {
        char *ret_str = NULL;
        PyArg_Parse(pRet, "s", &ret_str);
        std::cout << "python return value is : " << ret_str << std::endl;
    }
```

​	此处将 Python 的返回值通过 PyArg_Parse 函数解析为 C++ 的数据格式。s 表示返回值为字符串格式。

#### 3.3 C++调用Python类方法

​	通过C++传图像参数调用Python类方法实现图像检索。Python代码为NetVLAD图像检索算法，封装如下

```python
import cv2
import time
import os
import h5py
from netvlad_keras import NetVLADModel
import numpy as np
import tensorflow as tf

def arrayreset(array):
    a = array[:, 0:len(array[0] - 2):3]
    b = array[:, 1:len(array[0] - 2):3]
    c = array[:, 2:len(array[0] - 2):3]
    a = a[:, :, None]
    b = b[:, :, None]
    c = c[:, :, None]
    m = np.concatenate((a, b, c), axis=2)
    return m

def get_imlist(path):
    img_list = []
    for root, dirs, files in os.walk(path):
        for file in files:
            if file.endswith("jpg") or file.endswith("png") or file.endswith("jpeg"):
                # print(os.path.join(root, file))
                img_list.append(os.path.join(root, file))
    return img_list

def create_database(output,model):
    imgList = get_imlist("/home/zrc/backup/paramano_HZ/floor_3")
    feats = []
    names = []
    for imgPath in imgList:
        # name = imgPath.split("/")[-1]
        inim = cv2.imread(imgPath)
        inim = cv2.cvtColor(inim, cv2.COLOR_BGR2RGB)
        inim = cv2.resize(inim, (640, 360))
        # inim = cv2.resize(inim, (), interpolation=cv2.INTER_AREA)
        batch = np.expand_dims(inim, axis=0)
        # print(batch.shape)
        start = time.time()
        pred = model.predict(batch)[0]
        end = time.time()
        feats.append(pred)
        names.append(imgPath.encode())
        print("cost time: ", end - start)
    feats = np.array(feats)
    names = np.array(names)
    h5f = h5py.File(output, 'w')
    h5f.create_dataset("feats", data=feats)
    h5f.create_dataset("names", data=names)
    h5f.close()

class Retrieval():
    def __init__(self):
        self.model = NetVLADModel()
        self.model.load_weights("/home/zrc/CLionProjects/learn1_1/netvlad_weights.h5")
        self.model.build()

    def loadDatabase(self):
        print("read_database")
        databasePath = "/home/zrc/CLionProjects/learn1_1/database/feature.h5"
        h5f = h5py.File(databasePath, "r")
        self.feats = h5f["feats"][:]
        self.names = h5f["names"][:]
        h5f.close()

    def search(self,image):
        t1 = time.time()
        resultName = []
        query_img_raw = arrayreset(image)
        t2 = time.time()
        print("arrayreset cost time:",int((t2-t1)*1000),"ms")
        # print("query_img_rawshape()",query_img_raw.shape)
        query_img = cv2.cvtColor(query_img_raw, cv2.COLOR_BGR2RGB)
        # print("query_img()",query_img.shape)
        query_img = cv2.resize(query_img, (224, 224))
        # print("query_img()",query_img.shape)
        query_batch = np.expand_dims(query_img, axis=0)
        # print("query_batch()",query_batch.shape)
        query_descr = self.model.predict(query_batch)[0]
        t3 = time.time()
        print("query cost time:",int((t3-t2)*1000),"ms")
        # print("query_descr.shape()",query_descr.shape)
        score = np.dot(query_descr, np.array(self.feats).T)
        rankID = np.argsort(score)[::-1]
        rankScore = score[rankID]
        t4 = time.time()
        print("argsort cost time:",int((t4-t3)*1000),"ms")
        query_img_raw = cv2.copyMakeBorder(query_img_raw, 40, 40, 10, 10, cv2.BORDER_CONSTANT, value=(255,255,255))
        cv2.putText(query_img_raw,"Query Img",(10,30),1,2,(0,0,255),2)
        for i,ID in enumerate(rankID[:4]):
            name = self.names[ID].decode()
            result_img = cv2.imread(name)
            result_img = cv2.copyMakeBorder(result_img, 40, 40, 10, 10, cv2.BORDER_CONSTANT, value=(255,255,255))
            cv2.putText(result_img, "top %d"%(i+1), (10, 30), 1, 2, (255, 0, 0), 2)
            # print(i+1,":",name)
            resultName.append(name)
            query_img_raw = np.hstack((query_img_raw,result_img))
        print("=======================================================")
        cv2.resize(query_img_raw,(int(query_img_raw.shape[0]/3),int(query_img_raw.shape[1]/3)))
        cv2.imwrite("/home/zrc/CLionProjects/learn1_1/1.jpg",query_img_raw)
        return resultName
```

​	C++调用代码如下

```c++
//
// Created by zrc on 2021/2/22.
//
#include <Python.h>
#include "object.h"
#include <retrieval.h>
#include <opencv2/core/core.hpp>

int Rrtrieval::initPython() {
    Py_SetPythonHome(L"/home/zrc/anaconda3/envs/tf14");
    Py_Initialize();
    import_array();
    if(!Py_IsInitialized())
    {
        printf("Python init failed!\n");
        return -1;
    } else{
        PyRun_SimpleString("import sys");
        PyRun_SimpleString("import h5py");
        PyRun_SimpleString("sys.path.append('/home/zrc/CLionProjects/learn1_1')");
        pModule = PyImport_ImportModule("PyRetrieval");
        pDict = PyModule_GetDict(pModule);
        PyObject *pClass = PyDict_GetItemString(pDict, "Retrieval");
        assert(pClass != NULL);
        pInstance = PyObject_CallObject(pClass, NULL);
        assert(pInstance != NULL);
        return 1;
    }
}

void Rrtrieval::loadDatabase() {
//    pLoadDatabase = PyObject_GetAttrString(pModule, "loadDatabase");
    PyObject_CallMethod(pInstance, "loadDatabase", NULL);
}

void Rrtrieval::convertMat2Numpy(cv::Mat img) {
    int m, n;
    n = img.cols*3;
    m = img.rows;
    unsigned char *data = (unsigned  char*)malloc(sizeof(unsigned char) * m * n);
    int p = 0;
    for (int i = 0; i < m; i++)
    {
        for (int j = 0; j < n; j++)
        {
            data[p] = img.at<unsigned char>(i, j);
            p++;
        }
    }
    npy_intp Dims[2] = { m, n }; //给定维度信息
    PyArray = PyArray_SimpleNewFromData(2, Dims, NPY_UBYTE, data);
    delete data;
}

PyObject Rrtrieval::searchImg(cv::Mat img) {
    convertMat2Numpy(img);
    PyObject *ArgArray = PyTuple_New(1);
    PyTuple_SetItem(ArgArray, 0, PyArray);
    
    // s 表示字符串
    // i 表示整型变量
    // f 表示浮点数
    // O 表示一个 Python 对象
    pSearchResult=PyObject_CallMethod(pInstance, "search","O",ArgArray);
    
    int size = PyList_Size(pSearchResult);
    std::cout << "python return value list size is " << size << std::endl;
    for (int i = 0; i < size; ++i)
    {
        PyObject *pNameAge = PyList_GetItem(pSearchResult, i); // 获取 Python 返回值列表中的第 i 个元素
        char *ret = NULL;
        // 将 Python 的返回值转换为 CPP
        PyArg_Parse(pNameAge, "s", &ret);
        std::cout << "python return value is : " << ret << std::endl;
    }
}

void Rrtrieval::release() {
    if (pModule)
    {
        Py_DECREF(pModule);
    }
    Py_Finalize();
}
```

