# 软件API接口
## 软件设计框架及原理
## 环境选择
微纳智能软件需要用户在代码区域中选择所使用的仿真环境，在代码的**初始**及**结尾**使用特定环境代码语句，其中下述表示FDTD环境：
```
meep_start()
...
meep_end()
```
下述代码语句定义为lumerical环境：
```
lum_start()
...
lum_end()
```
下述代码语句定义为CST环境：
```
cst_start()
...
cst_end()
```
## 0. 全局定义
模型仿真需要定义全局变量
* nfreq 用来定义频率采样点，默认值为200
* wl_max 用来定义仿真所用波长最大值，默认值为1600nm
* wl_min 用来定义仿真所用波长最小值，默认值为1100nm
* T_z meep中监视器末端
* resolution
* auto_shutoff_min

## 1. 模型构建



## 2. 源设置




## 3. 监视器
仿真所使用监视器有两类：功率监视器及电场监视器。
## 3.1 功率监视器
add_monitor(center, size, name, color, opacity)来定义功率监视器
* center及size参数需写为(x,y,z)三维区域，来定义监视器中心及大小
* name参数用来定义监视器的名称，方便后续提取监视器结果
* color参数用来定义监视器展示颜色，默认为red
* opacity参数用来定义监视器展示透明度，默认为0.4

*备注: 用户必需定义center，size，name参数来建立监视器，否则无法加载*

## 3.2 电场监视器
add_monitor_field(center, size, name, color, opactity)来定义功率监视器
* center及size参数需写为(x,y,z)三维区域，来定义监视器中心及大小
* name参数用来定义监视器的名称，方便后续提取监视器结果
* nfreq参数来定义监视器的频率点，默认为200
* color参数用来定义监视器展示颜色，默认为red
* opacity参数用来定义监视器展示透明度，默认为0.4

*备注: 用户必需定义center，size，name参数来建立监视器，否则无法加载*

## 4. 仿真区域
add_sim_area(center, size)来定义模型的仿真区域，



## 5. 仿真运行



## 6. 结果提取



## 7. 性能图保存


