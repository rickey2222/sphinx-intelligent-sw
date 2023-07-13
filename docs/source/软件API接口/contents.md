# 软件API接口
## 软件设计框架及原理
基于智能算法的微纳光学逆向设计软件平台，包括微纳光学正向及逆向仿真功能。微纳智能软件可进行数值仿真，并且能够实现与主流商用光电子软件-Lumerical及CST联合仿真。用户可在软件代码区域使用统一API设计FDTD、Lumerical及CST环境所使用的模型。
软件是基于容器建立后端服务，通过调度容器对代码区域上下文的分析，将对应的算法片段发送到远程的执行软件执行，并且收集结果，将建模及结果展示在前端软件中。其中前端与调度容器通过HTTP协议传送信息，不同算法容器的计算结果保存在MongoDB中。
![容器架构.png](https://s2.loli.net/2023/07/13/GBoaIVu5ZgLMS2x.png)
<center> 图4.1 软件架构原理图 </center>

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


