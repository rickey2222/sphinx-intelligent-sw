# 软件API接口
## 软件设计框架及原理



## 仿真
### 环境选择
微纳光学智能设计软件中，在软件代码区域根据仿真模型选择适配的运行环境。分别在建模代码的**初始**及**结尾**部分调用环境代码：
+ FDTD环境：
```
sim_start(backend='meep')
...
sim_end(backend='meep')
```
+ lumerical环境：
```
sim_start(backend='lum')
...
sim_end(backend='lum')
```
+ CST环境：
```
sim_start(backend='cst')
...
sim_end(backend='cst')
```
### 全局定义
模型仿真需要定义全局变量
* nfreq[数值类型]：频率采样点，默认值为200
* wl_max[数值类型]：仿真所用波长最大值，默认值为1600nm
* wl_min[数值类型] ：仿真所用波长最小值，默认值为1100nm
* T_z meep中监视器末端
* resolution
* auto_shutoff_min

### 模型构建



### 源设置




### 监视器
仿真所使用监视器有两类：功率监视器及电场监视器。
#### 功率监视器
add_monitor(center, size, name, color, opacity)定义功率监视器
* center及size参数需写为(x,y,z)三维数组，表示监视器中心及大小
* name参数用来定义监视器的名称，方便后续提取监视器结果
* color参数用来定义监视器展示颜色，默认为red
* opacity参数用来定义监视器展示透明度，默认为0.4

*备注: 用户必需定义center，size，name参数来建立监视器，否则无法加载*

#### 电场监视器
add_monitor_field(center, size, name, color, opactity)定义功率监视器
* center及size参数需写为(x,y,z)三维数组，表示监视器中心及大小
* name参数用来定义监视器的名称，方便后续提取监视器结果
* nfreq参数来定义监视器的频率点，默认为200
* color参数用来定义监视器展示颜色，默认为red
* opacity参数用来定义监视器展示透明度，默认为0.4

*备注: 用户必需定义center，size，name参数来建立监视器，否则无法加载*

### 仿真区域
add_sim_area(center, size)定义模型的仿真区域



### 仿真运行



### 结果提取



### 性能图保存

## 教程/波导仿真
## 教程/微纳偏振结构


