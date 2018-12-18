# 性能检测指标
* 启动时间
* CPU
* MEM
* 页面渲染时间，刷新帧率FPS
* 网络请求时间
* 耗电功率
* UI阻塞次数（主线程阻塞超过400ms，用户会感知到卡顿）

# 性能检测方法
期望

* 方便接入
* 可生成性能报告
* 可持续化
* 可保证数据收集的精准性

## Instrument


思路： 
 
 * 方案一：使用instruments进行测试 -- 缺点：官方提供，无法自动采集
  * 使用AppleScript自动化实现instruments的启动，操作和性能数据的读取。  
 * 方案二：使用官方提供的API写脚本进行性能采集

 ### instruments的自动化获取
 
 ### 脚本开发 
 
 1. 获取MEM ／CPU

 ``` 
 #import <mach/mach.h>

/**
 *  获取内存
 */
- (NSString *)get_memory {
    int64_t memoryUsageInByte = 0;
    task_vm_info_data_t vmInfo;
    mach_msg_type_number_t count = TASK_VM_INFO_COUNT;
    kern_return_t kernelReturn = task_info(mach_task_self(), TASK_VM_INFO, (task_info_t) &vmInfo, &count);
    if(kernelReturn == KERN_SUCCESS) {
        memoryUsageInByte = (int64_t) vmInfo.phys_footprint;
        NSLog(@"Memory in use (in bytes): %lld", memoryUsageInByte);
    } else {
        NSLog(@"Error with task_info(): %s", mach_error_string(kernelReturn));
    }

    double mem = memoryUsageInByte / (1024.0 * 1024.0);
    NSString *memtostring ;
    memtostring = [NSString stringWithFormat:@"%.1lf",mem];

    return memtostring;
}


/**
 * 获取cpu
 */
- (NSString *) get_cpu{
    kern_return_t kr;
    task_info_data_t tinfo;
    mach_msg_type_number_t task_info_count;

    task_info_count = TASK_INFO_MAX;
    kr = task_info(mach_task_self(), TASK_BASIC_INFO, (task_info_t)tinfo, &task_info_count);
    if (kr != KERN_SUCCESS) {
        return [ NSString stringWithFormat: @"%f" ,-1];
    }

    task_basic_info_t      basic_info;
    thread_array_t         thread_list;
    mach_msg_type_number_t thread_count;

    thread_info_data_t     thinfo;
    mach_msg_type_number_t thread_info_count;

    thread_basic_info_t basic_info_th;
    uint32_t stat_thread = 0; // Mach threads

    basic_info = (task_basic_info_t)tinfo;

    // get threads in the task
    kr = task_threads(mach_task_self(), &thread_list, &thread_count);
    if (kr != KERN_SUCCESS) {
        return [ NSString stringWithFormat: @"%f" ,-1];
    }
    if (thread_count > 0)
        stat_thread += thread_count;

    long tot_sec = 0;
    long tot_usec = 0;
    float tot_cpu = 0;
    int j;

    for (j = 0; j < thread_count; j++)
    {
        thread_info_count = THREAD_INFO_MAX;
        kr = thread_info(thread_list[j], THREAD_BASIC_INFO,
                         (thread_info_t)thinfo, &thread_info_count);
        if (kr != KERN_SUCCESS) {
            tot_cpu = -1;
            //return -1;
        }

        basic_info_th = (thread_basic_info_t)thinfo;

        if (!(basic_info_th->flags & TH_FLAGS_IDLE)) {
            tot_sec = tot_sec + basic_info_th->user_time.seconds + basic_info_th->system_time.seconds;
            tot_usec = tot_usec + basic_info_th->user_time.microseconds + basic_info_th->system_time.microseconds;
            tot_cpu = tot_cpu + basic_info_th->cpu_usage / (float)TH_USAGE_SCALE * 100.0;
        }

    } // for each thread

    kr = vm_deallocate(mach_task_self(), (vm_offset_t)thread_list, thread_count * sizeof(thread_t));
    assert(kr == KERN_SUCCESS);

    NSString *tostring = nil ;
    tostring = [ NSString stringWithFormat: @"%.1f" ,tot_cpu];
    NSLog (@"performance  cpu:%@",tostring);

    return tostring;}```
    
  
 2. 获取页面VC
  
  ```
   收集页面信息，与上述性能指标一一对应
   /**
 *获取当前vc
 */
 - (UIViewController *) get_vc {
    UIWindow *keyWindow = [UIApplication sharedApplication].keyWindow;
    __weak typeof(self) weakSelf = self;
    dispatch_async(dispatch_get_main_queue(), ^{
        if ([keyWindow.rootViewController isKindOfClass:[UITabBarController class]]) {
            UITabBarController *tab = (UITabBarController *)keyWindow.rootViewController;
            UINavigationController *nav = tab.childViewControllers[tab.selectedIndex];
            DDContainerController *content = [nav topViewController];
            weakSelf.vc = [content contentViewController];
        }
    });
    return self.vc;
}
  
   ``` 

3. 获取设备信息

```
/*
 *获取设备名称
 */
- (NSString *) get_devicesName {
    NSString *devicesName = [UIDevice currentDevice].name; //设备名称
    NSLog(@"performance   devicesName:%@", devicesName);
    return devicesName;

}

/*
 *获取系统版本
 */
- (NSString *) get_systemVersion{
    NSString *systemVersion = [UIDevice currentDevice].systemVersion; //系统版本
    NSLog(@"performance   version:%@", systemVersion);
    return systemVersion;
}

/*
 *获取设备idf
 */
- (NSString *) get_idf {
    NSString *idf = [UIDevice currentDevice].identifierForVendor.UUIDString;
    NSLog(@"performance   idf:%@", idf);
    return idf;

}
数据拼接
最终要把内存、cpu等数据拼接成字典的形式,方便输出查看

输出log日志的数据格式

{
    "cpu": "0.4",
    "fps": "60 FPS",
    "version": "11.2",
    "appname": "xxxxxx",
    "battery": "-100.0",
    "appversion": "5.0.4",
    "time": "2018-09-07 11:45:24",
    "memory": "141.9",
    "devicesName": "xxxxxx",
    "vcClass": "DDAlreadPaidTabListVC",
    "idf": "8863F83E-70CB-43D5-B6C7-EAB85F3A2AAD"
}
```
参考：[脚本自动化](https://testerhome.com/topics/16064)


