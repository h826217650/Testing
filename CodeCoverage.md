#代码覆盖率

## 1 why
* 为什么进行代码覆盖率测试  

  发现未被测试覆盖的代码的手段
## 2 What
* 什么是代码覆盖率测试，其原理是什么？  
### 2.1 覆盖率工具
![](/Users/huxiaoyan/Desktop/代码覆盖率方式.jpg)

### 2.2 工作流程
1. 对java字节码进行查桩（on-the-fly 和 offine两种方式）
2. 执行测试用例，收集执行轨迹，dump到内存
3. 数据处理器根据用例执行轨迹和代码结构生成覆盖率
4. 覆盖率测试报告可视化呈现（html）

### 2.3 覆盖率实现原理
代码覆盖率工具采用字节码插桩模式实现。
_on-the-fly插桩 java agent(jacoco采用该方式)_  

*  jvm 通过java agent参数指定特定的jar文件来启动instrumentation的代理程序   
*  代理程序在装载class文件前判断是否已经修改了该文件，如没有则把探针插入class文件中  
*  代码覆盖率就可以在JVM执行代码的时候实时获取   

_on-the-fly插桩 class loader(emma采用该方式)_
* 自定义class loader实现类装载策略，在类加载前插入探针到class文件

> 使用 Instrumentation，开发者可以构建一个独立于应用程序的代理程序（Agent），用来监测和协助运行在 JVM 上的程序，甚至能够替换和修改某些类的定义。
[JAVA Instrumentation介绍](https://www.ibm.com/developerworks/cn/java/j-lo-jse61/index.html)

## 3 How
* 如何进行代码覆盖率部署

### 3.1 Jacoco(java code coverage)


 