# 执行性能测试之前需了解  

* 项目框架
* 使用的编程语言：  

  * java
  * native
  * object-C
  * C／C++

 
* 使用的系统框架： 

  * tomcat
  * nginx
  * servlet 

* 数据库：mysql

# 执行性能测试
* 根据项目，制定需要关注的性能指标 

  * CPU
  * MEM
  * 耗电量
  * 帧率
  * 发烫情况
  * 最大QPS
  * 容灾能力 

* 根据业务场景，制定性能测试策略

  * 基准测试  
    正常情况下的性能指标输出 （CPU／MEM／FPS／耗电量／发烫）
  * 负载测试  
    查找系统瓶颈，如最大QPS
  * 压力测试   
    极端情况下的测试，让系统崩溃，看恢复能力及响应时间
    
    
* 执行过程 
  * 测试数据准备：如何保证测试数据的一致性
  * 测试框架选型：如何选择／搭建合适的测试系统
  * 测试脚本构建：如何根据业务逻辑构建完整的测试脚本case
  * 测试数据分析：如何有效利用测试数据进行分析优化
    
    