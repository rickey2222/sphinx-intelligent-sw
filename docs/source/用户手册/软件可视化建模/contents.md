# 智能软件可视化建模

## 1.可视化建模区块介绍

可视化建模需要用户提交模型python代码到网页的代码区中，前端采用CodeMirror在线代码编辑器使得用户可以在对应的算法环境中定义仿真模型。同时用户自定义模型会通过Three.js框架实现3D效果，通过编辑器中的模型定义可在模型展示区中XY view、YZ view、XZ view二维平面以及Perspective View三维视角显示模型的视图，并且通过鼠标可拖拽模型方向及显示大小。
![图片1.png](https://s2.loli.net/2022/12/23/tYEC3496qyFvzn8.png)
<center> 图1.1 可视化建模 </center>

现阶段软件可提供基于MEEP的FDTD算法以及与商业软件联动的CST算法背景，需要用户切换算法环境。



## 2. 可视化建模注意事项
因软件还在开发阶段，用户搭建模型需按照已给定的实例进行改写：
* 用户在代码区定义器件后需点击上方运行按钮，软件会进行模型搭建，返回的结果可在解释去输出窗口读取，相应的性能结果保存在对应的性能图中，同时模型相关要素展示在要素预览区。
* 模型解析完成后解释区会有FINISHED输出，如若计算中想停止运行可点击代码上方的暂停按钮。

## 3. 代码示例说明


### 3.1 meep 示例

运行该示例（器件库中的meta_em5)时，网页端的算法环境选择FDTD，
以下是调用所需要的依赖包：

    import time
    import meep as mp
    import numpy as np
    from matplotlib import pyplot as plt
 

由于现在还没有统一接口，目前相关前端展示代码依然编写在编辑器中，后期会封装有统一的接口。
 
    tree = ModelTree('Model')  # 这是在前端创建要素预览
 

#### 3.1.1 参数定义
建立仿真模型所需要的变量参数：

    parameter = {'dmet': 0.27,
             'wl_max': 2.5,
             'wl_min': 1,
             'dp': 0.8,
             'dsub': 2,
             'n1': 1.45,
             'n2':  3.67
             }
    par = parameter
    dp = par["dp"]
    dsub = par["dsub"]
    n1 = par["n1"]
    n2 = par["n2"]
    fmax = 1 / par['wl_min']
    fmin = 1 / par['wl_max']
    fcen = (fmin + fmax) / 2
    df = fmax - fmin
    dpml = 1 / fcen / 2
    nfreq = 200
    starttime = time.time()
    resolution = 40
    cell_size = mp.Vector3(dp, dp, 2 + 2 * dpml)  # only Z-axis with PML
    k_point = mp.Vector3(0, 0, 0)
    pml_layers = [mp.PML(thickness=dpml, direction=mp.Z)]
    pt = mp.Vector3(0, 0, -0.6)
 
 
#### 3.1.2 建立仿真模型

meep中添加仿真所需要的模型结构，mp.Block是添加一个长方体模型，mp.Cylinder 是添加一个圆柱体，其中center属性为中心坐标点，size属性为中心坐标延展到x,y,z的尺寸，radius属性为半径，height属性为高度。关于更多形状的建模函数参考 <https://meep.readthedocs.io/en/latest/Python_User_Interface/>

    c1 = mp.Block(size=mp.Vector3(dp, dp, dsub),
              center=mp.Vector3(0, 0, -1),
              material=mp.Medium(index=n1)) # 添加一个材料为n1的长方体，中心坐标为(0，0，-1),x,y方向的长度为dp,z方向的长度为dsub

    c2 = mp.Block(size=mp.Vector3(0.6, 0.6, 0.27),
                center=mp.Vector3(0, 0, 0.135),
                material=mp.Medium(index=n2))

    c3 = mp.Cylinder(radius=0.2, center=(0.3, 0.3, 0.135), height=0.27) # 材料折射率默认为1
    c4 = mp.Cylinder(radius=0.2, center=(0.3, -0.3, 0.135), height=0.27)
    c5 = mp.Cylinder(radius=0.2, center=(-0.3, 0.3, 0.135), height=0.27)
    c6 = mp.Cylinder(radius=0.2, center=(-0.3, -0.3, 0.135), height=0.27)
    geometry = [c1, c2, c3, c4, c5, c6]

    
    # 以下是在前端需要展示的模型结构代码： （后续会统一接口可省略，前端展示模型结构坐标与仿真模型坐标同步）

    c1_id = create_model(size=(dp, dp, dsub), position=(0, 0, -1), type='Box', color='gray', opacity=0.5) # 创建模型id，颜色可通过color 属性自定义，模型透明度可通过opacity属性自定义，type属性定义模型类型
    node1 = tree.add_node('c1 - Block') # 创建要素预览

    c2_id = create_model(size=(0.6, 0.6, 0.27), position=(0, 0, 0.135), type='Box', color='Green')
    node2 = tree.add_node('c2 - Block')

    c3_id = create_model(size=(0.2, 0.2, 0.27, 40), position=(0.3, 0.3, 0.135), type='Cylinder', color='White' )
    node3 = tree.add_node('c3 - Cylinder')

    c4_id = create_model(size=(0.2, 0.2, 0.27, 40), position=(0.3, -0.3, 0.135), type='Cylinder', color='White')
    node4 = tree.add_node('c4 - Cylinder')

    c5_id = create_model(size=(0.2, 0.2, 0.27, 40), position=(-0.3, 0.3, 0.135), type='Cylinder', color='White')
    node5 = tree.add_node('c5 - Cylinder')

    c6_id = create_model(size=(0.2, 0.2, 0.27, 40), position=(-0.3, -0.3, 0.135), type='Cylinder', color='White')
    node6 = tree.add_node('c6 - Cylinder')


#### 3.1.3 添加光源、仿真区域、监视器

光源是由中心坐标为(0, 0, 0.65)构成的沿xy平面的矩形框，component为光源的分量参数。creat_model()函数是在前端展示的模型结构函数，可自定义颜色，透明度，位置坐标与前面仿真所建立的参数一直即可，后续会统一接口省略多余的参数定义，以下遇到的同样的定义类似。  

    sources = [mp.Source(mp.GaussianSource(fcen, fwidth=df),
                        component=mp.Ey,
                        center=mp.Vector3(0, 0, 0.65),
                        size=mp.Vector3(0.8, 0.8, 0))]

    # 以下是在前端需要展示的光源结构代码： （后续会统一接口可省略，前端展示模型结构坐标与仿真模型坐标同步）
    source_id = create_model(position=(0, 0, 0.65), size=(0.8, 0.8, 0.01), type='Box', line='true', color='Red')
    arrow_id = create_model(source=(0, 0, 0.65), target=(0, 0, 0.1), type='Arrow', color='Red')
    node7 = tree.add_node('Light Source', 'source')


添加fdtd仿真区域，仿真区域主要是由cell大小区域的长方体构成的，中心坐标为（0,0,0），x方向的长度为dp，y方向的长度为dp，z方向的长度为2+2*dpml。

    sim = mp.Simulation(resolution=resolution,
                        cell_size=cell_size,
                        boundary_layers=pml_layers,
                        k_point=k_point,
                        default_material=mp.Medium(index=1),
                        sources=sources, )

    sim_id = create_model(position=(0, 0, 0), size=(dp, dp, 2 + 2 * dpml), type='Box', color='Orange', opacity=0.4)
    node8 = tree.add_node('Sim Area', 'area')


添加入射监视器区域

    inc_fr = mp.FluxRegion(center=(0, 0, 0.8), size=mp.Vector3(0.8, 0.8, 0))
    inc = sim.add_flux(fcen, df, nfreq, inc_fr)

    monitor_id = create_model(position=(0, 0, 0.8), size=(0.8, 0.8, 0.01), type='Box', line='true', color='Red')
    node9 = tree.add_node('Monitor', 'area')


添加反射监视器区域

    refl_fr = mp.FluxRegion(center=(0, 0, 0.8), size=mp.Vector3(0.8, 0.8, 0))
    refl = sim.add_flux(fcen, df, nfreq, refl_fr)

    refl_id = create_model(position=(0, 0, 0.8), size=(0.8, 0.8, 0.01), type='Box', line='true', color='Red')
    node10 = tree.add_node('Reflection Area', 'area')

 添加透射监视器区域

    tran_fr = mp.FluxRegion(center=mp.Vector3(0, 0, -0.5), size=mp.Vector3(0.8, 0.8, 0))
    tran = sim.add_flux(fcen, df, nfreq, tran_fr)

    tran_id = create_model(position=(0, 0, -0.5), size=(0.8, 0.8, 0.01), type='Box', line='true', color='Yellow')
    node11 = tree.add_node('Transparent Area', 'area')

#### 3.1.4 仿真运行
因为该meta示例为了计算反射率，透射率、吸收率谱线图，所以运行了两次仿真，第一次仿真是为了计算反射率，没有添加任何几何结构，第二次仿真添加了我们设计的超表面结构，最终仿真得出光谱图。其中，device.sav_fig()函数

    # 第一次运行
    sim.run(until_after_sources=mp.stop_when_fields_decayed(50, mp.Ey, pt, 5e-8))
    input_flux = mp.get_fluxes(inc)
    refl_data = sim.get_flux_data(refl)
    sim.reset_meep()

    sim = mp.Simulation(resolution=resolution,
                        cell_size=cell_size,
                        boundary_layers=pml_layers,
                        geometry=geometry,
                        default_material=mp.Medium(index=1),
                        k_point=k_point,
                        sources=sources, )

    refl = sim.add_flux(fcen, df, nfreq, refl_fr)
    sim.load_minus_flux_data(refl, refl_data)

    # 添加透射监视器区域
    tran_fr = mp.FluxRegion(center=mp.Vector3(0, 0, -0.5), size=mp.Vector3(0.8, 0.8, 0))
    tran = sim.add_flux(fcen, df, nfreq, tran_fr)
    tran_id = create_model(position=(0, 0, -0.5), size=(0.8, 0.8, 0), type='Box', line='true', color='Red')
    node12 = tree.add_node('Flux Region', 'area')

    # 输出xz截面图
    f2 = plt.figure(dpi=120)
    sim.plot2D(ax=f2.gca(), output_plane=mp.Volume(center=mp.Vector3(), size=mp.Vector3(dp, 0, 2 + 2 * dpml)))
    plt.savefig('2D structrue(XZ).png')
    device.save_fig('2D structrue(XZ).png', file_path='2D structrue(XZ).png') # 这个是保存到前端命令，后续统一接口

    # 输出xy截面图
    f2 = plt.figure(dpi=120)
    sim.plot2D(ax=f2.gca(), output_plane=mp.Volume(center=mp.Vector3(), size=mp.Vector3(dp, dp, 0)))
    plt.savefig('2D structrue(XY).png')
    device.save_fig('2D structrue(XY).png', file_path='2D structrue(XY).png') # 这个是保存到前端命令，后续统一接口

    # 第二次运行
    sim.run(until_after_sources=mp.stop_when_fields_decayed(50, mp.Ey, pt, 5e-8))

    refl_flux = mp.get_fluxes(refl)
    tran_flux = mp.get_fluxes(tran)
    flux_freqs = mp.get_flux_freqs(refl)

#### 3.1.5 后处理数据

    ez_data = sim.get_array(mp.Ez, mp.Volume(center=mp.Vector3(), size=mp.Vector3(dp, dp, 0)))  # 从仿真结果中提取某一截面的Ez电场分布图数据

    plt.imshow(np.flipud(np.transpose(ez_data)), interpolation='spline36', cmap='RdBu', alpha=0.9)
    a = np.flipud(np.transpose(ez_data))
    np.savetxt("data_meta5_Ez_field.txt", a)  # 导出Ez电场分布图二维数据
    plt.savefig("meta5_Ez_field.png")  # Ez电场分布图
    device.save_fig('meta5_Ez_field.png', file_path='meta5_Ez_field.png')

    wl = []
    Rs = []
    Ts = []
    endtime = time.time()
    total_time = endtime - starttime
    print(f"Total Running Time : {total_time} (s)")

    for i in range(nfreq):
        wl = np.append(wl, 1 / flux_freqs[i])
        Rs = np.append(Rs, refl_flux[i] / input_flux[i])
        Ts = np.append(Ts, -tran_flux[i] / input_flux[i])
    data = np.zeros((len(wl), 4))
    data[:, 0] = wl  # 波长 对应x轴
    data[:, 1] = Rs  # 反射率 对应y轴
    data[:, 2] = Ts  # 透射率 对应y轴
    data[:, 3] = 1 - Rs - Ts  # 吸收率 对应y轴
    np.savetxt("data_meta5_refl_tran.txt", data)  # 导出光谱图(透射率、反射率、吸收率)的数据

    plt.figure()
    plt.plot(wl, Rs, 'b', label='reflectance')
    plt.plot(wl, Ts, 'r', label='transmittance')
    plt.plot(wl, 1 - Rs - Ts, 'g', label='loss')
    plt.xlabel("wavelength (μm)")
    plt.legend(loc="upper right")
    plt.savefig('meta5_refl_tran.png')
    device.save_fig('meta5_refl_tran.png', file_path='meta5_refl_tran.png')




### 3.2 lumerical 示例

导入所需要的依赖包

    import sys, os, math
    import numpy as np
    import matplotlib.pyplot as plt
    sys.path.append("C:\\Program Files\\Lumerical\\v202\\api\\python\\")  # Default windows lumapi path
    # sys.path.append(os.path.dirname(__file__))  # Current directory
    import lumapi

定义的仿真模型函数，该示例仿真的是波导器件。

    def sim_wav(fdtd, name):
        ## 创建预览要素
        tree = ModelTree('Model')
        # 添加波导器件:中心坐标为（0，0，0），尺寸为（10，0.5，0.22）的长方体 ，材料折射率为3.45 单位：μm
        fdtd.addrect();
        fdtd.set('name', 'wav');
        fdtd.set('x span', 10e-6);
        fdtd.set('y span', 0.5e-6);
        fdtd.set('z span', 0.22e-6);
        fdtd.set('x', 0);
        fdtd.set('y', 0);
        fdtd.set('z', 0);
        fdtd.set('material', "<Object defined dielectric>");
        fdtd.set('index', 3.45)
        c1_id = create_model(size=(10,  0.5, 0.22), position=(0, 0, 0), type='Box', color='green',opacity=0.8)
        node1 = tree.add_node('wav')

        # 添加 fdtd：中心坐标为（0，0，0），尺寸为（6，3.4，4.22）的长方体框，单位：μm
        fdtd.addfdtd();
        fdtd.set('dimension', '3D');
        fdtd.set('x span', 6e-6);
        fdtd.set('y span', 3.4e-6);
        fdtd.set('z span', 4.22e-6)
        fdtd.set('x', 0);
        fdtd.set('y', 0);
        fdtd.set('z', 0)
        fdtd.set("index", 1.44)

        # 前端创建fdtd仿真域结构
        create_model(position=(0, 0, 0), size=(6, 3.4, 4.22), type='Box', color='Orange',opacity=0.8)
        tree.add_node('fdtd', 'area')

        # add  mode sources
        fdtd.addmode(name="TE", mode_selection=2, x=-2.5e-6, y=0, z=0, y_span=2e-6, z_span=4.22e-6, \
                    wavelength_start=1.26e-6, wavelength_stop=1.36e-6, number_of_trial_modes=20)
        
        # 前端创建光源结构
        monitor_id = create_model(position=(-2.5, 0, 0), size=(0, 2, 4.22), type='Box', line='true', color='White')
        arrow_id = create_model(source=(-2.5, 0, 0), target=(0.5, 0, 0), type='Arrow', color='Red')
        node7 = tree.add_node('Light Source', 'source')

        # add field monitor
        fdtd.addpower(name="T", monitor_type=5, x=2.5e-6, y=0, z=0, y_span=1.5e-6, z_span=4.22e-6)
        fdtd.addprofile(name="E_field", monitor_type=7, x=0, y=0, z=0, x_span=6e-6, y_span=3.4e-6)

        # 前端创建监视器结构
        create_model(position=(2.5, 0, 0), size=(0, 1.5, 4.22), type='Box', color='Yellow',opacity=0.8)
        tree.add_node('T', 'area')
        create_model(position=(0, 0, 0), size=(6, 3.4, 0), type='Box', color='Yellow',opacity=0.8)
        tree.add_node('E_field', 'area')

        # 添加模式分析器
        fdtd.addmodeexpansion();
        fdtd.set("name", "mode_exp1");
        fdtd.set("mode selection", "fundamental TE mode");
        fdtd.set("x", 2.5e-6);
        fdtd.set("y", 0);
        fdtd.set("z", 0);
        fdtd.set("y span", 1.5e-6);
        fdtd.set("z span", 4.22e-6);
        # set the field monitor to be used by the expansion monitor
        fdtd.select("mode_exp1");
        fdtd.setexpansion("out_1", "T");
        create_model(position=(2.5, 0, 0), size=(0, 1.5, 4.22), type='Box', color='Yellow',opacity=0.8)
        tree.add_node('mode_exp1', 'area')

        # save
        fdtd.save("D:\\%s" % name)

        # run
        fdtd.run()

        # 处理数据
        expansion_1 = fdtd.getresult("mode_exp1", "expansion for out_1")
        T = fdtd.getattribute(expansion_1, "T_forward")
        f = fdtd.getdata("T", "f")
        fig, ax = plt.subplots(1, 1)
        plt.plot(3 * 1e8 / f * 1e6, T)
        # np.savetxt("wav_T.txt",T)
        plt.savefig("wav_T.png")
        device.save_fig('wav_T', file_path="wav_T.png")

        E_total = ["Ex", "Ey", "Ez"]

        for i in E_total:
            fig, ax = plt.subplots(1, 1)
            E = fdtd.getdata("E_field", i)
            x = fdtd.getdata("E_field", "x")
            y = fdtd.getdata("E_field", "y")
            data_E = np.squeeze(np.array(np.real(E)))
            plt.imshow(data_E[:, :, 0].T, origin="lower")
            plt.colorbar()
            ax.set_title(f"{i}_field")
            # np.savetxt(f"wav_{i}.txt", data_E[:,:,0].T)
            plt.savefig(f"wav_{i}.png")
            device.save_fig(f"wav_{i}", file_path=f"wav_{i}.png")   #前端保存图片
        # plt.show()


    fdtd = lumapi.FDTD(hide=False)  #调lumeircal仿真软件
    sim_wav(fdtd, "waveguide")
    fdtd.close()


### 3.3 cst 示例

以十字形吸收器为例。

Step1: 导入所需要的依赖包：

```
import sys

cst_lib_path = r"D:\Program Files (x86)\CST_Studio_Suite_2020\AMD64\python_cst_libraries"
sys.path.append(cst_lib_path)

import cst
from cst import interface, results

import time
import os
os.environ['KMP_DUPLICATE_LIB_OK']='TRUE'

import numpy as np
import matplotlib.pyplot as plt

from cst.interface import Project
import math
from cst import results
import numpy as np
```



Step2: 定义用到的函数：

+ 基本环境设置

```
def setup_base(modeler: Project.modeler):

    # setup units
    vba = '''
    With Units
        .Geometry "um"
        .Frequency "THz"
        .Voltage "V"
        .Resistance "Ohm"
        .Inductance "H"
        .TemperatureUnit  "Kelvin"
        .Time "ns"
        .Current "A"
        .Conductance "Siemens"
        .Capacitance "F"
    End With
    '''
    modeler.add_to_history('setup units', vba)

    # set the wavelength range
    vba = '''
    Solver.WavelengthRange "2.5", "6"
    '''
    modeler.add_to_history('setup frequency range', vba)

    # setup background
    vba = '''
    Plot.DrawBox True
    With Background
         .Type "Normal"
         .Epsilon "1.0"
         .Mu "1.0"
         .Rho "1.204"
         .ThermalType "Normal"
         .ThermalConductivity "0.026"
         .HeatCapacity "1.005"
         .XminSpace "0.0"
         .XmaxSpace "0.0"
         .YminSpace "0.0"
         .YmaxSpace "0.0"
         .ZminSpace "0.0"
         .ZmaxSpace "0.0"
    End With
    '''
    modeler.add_to_history('setup background', vba)

    # define Floquet port boundaries
    vba = '''
    With FloquetPort
         .Reset
         .SetDialogTheta "0"
         .SetDialogPhi "0"
         .SetSortCode "+beta/pw"
         .SetCustomizedListFlag "False"
         .Port "Zmin"
         .SetNumberOfModesConsidered "2"
         .Port "Zmax"
         .SetNumberOfModesConsidered "2"
    End With
    '''
    modeler.add_to_history('set Floquet port boundaries', vba)

    # define parameters exist
    vba = '''
    MakeSureParameterExists "theta", "0"
    SetParameterDescription "theta", "spherical angle of incident plane wave"
    MakeSureParameterExists "phi", "0"
    SetParameterDescription "phi", "spherical angle of incident plane wave"
    '''
    modeler.add_to_history('set parameters exist', vba)

    # define boundaries, the open boundaries define floquet port
    vba = '''
    With Boundary
         .Xmin "unit cell"
         .Xmax "unit cell"
         .Ymin "unit cell"
         .Ymax "unit cell"
         .Zmin "expanded open"
         .Zmax "expanded open"
         .Xsymmetry "none"
         .Ysymmetry "none"
         .Zsymmetry "none"
         .XPeriodicShift "0.0"
         .YPeriodicShift "0.0"
         .ZPeriodicShift "0.0"
         .PeriodicUseConstantAngles "False"
         .SetPeriodicBoundaryAngles "theta", "phi"
         .SetPeriodicBoundaryAnglesDirection "inward"
         .UnitCellFitToBoundingBox "True"
         .UnitCellDs1 "0.0"
         .UnitCellDs2 "0.0"
         .UnitCellAngle "90.0"
    End With    
    '''
    modeler.add_to_history('set boundaries', vba)

    # set tet mesh as default
    vba = '''
    With Mesh
         .MeshType "Tetrahedral"
    End With
    '''
    modeler.add_to_history('set mesh', vba)

    # FD solver excitation with incoming plane wave at Zmax
    vba = '''
    With FDSolver
         .Reset
         .Stimulation "List", "List"
         .ResetExcitationList
         .AddToExcitationList "Zmax", "TE(0,0);TM(0,0)"
         .LowFrequencyStabilization "False"
    End With
    '''
    modeler.add_to_history('set FD solver', vba)

    # change problem type
    vba = '''
    ChangeProblemType "Optical"
    '''
    modeler.add_to_history('change problem type', vba)

    vba = '''
    With MeshSettings
         .SetMeshType "Tet"
         .Set "Version", 1%
    End With
    '''
    modeler.add_to_history('MeshSettings', vba)

    vba = '''
    With Mesh
         .MeshType "Tetrahedral"
    End With
    '''
    modeler.add_to_history('cset meshtype', vba)

    # set the solver type
    vba = '''
    ChangeSolverType("HF Frequency Domain")
    '''
    modeler.add_to_history('set the solver type', vba)

    # set component
    vba = '''
    Component.New "component1" 
    '''
    modeler.add_to_history('set component', vba)
```

+ 定义金的Drude模型

```
def gold_drude(modeler):
    vba = '''
    With Material 
         .Reset 
         .Name "gold_drude"
         .Folder ""
         .Rho "0.0"
         .ThermalType "Normal"
         .ThermalConductivity "0"
         .SpecificHeat "0", "J/K/kg"
         .DynamicViscosity "0"
         .Emissivity "0"
         .MetabolicRate "0.0"
         .VoxelConvection "0.0"
         .BloodFlow "0"
         .MechanicsType "Unused"
         .FrqType "all"
         .Type "Normal"
         .MaterialUnit "Frequency", "THz"
         .MaterialUnit "Geometry", "um"
         .MaterialUnit "Time", "ns"
         .MaterialUnit "Temperature", "Kelvin"
         .Epsilon "1"
         .Mu "1"
         .Sigma "0"
         .TanD "0.0"
         .TanDFreq "0.0"
         .TanDGiven "False"
         .TanDModel "ConstTanD"
         .EnableUserConstTanDModelOrderEps "False"
         .ConstTanDModelOrderEps "1"
         .SetElParametricConductivity "False"
         .ReferenceCoordSystem "Global"
         .CoordSystemType "Cartesian"
         .SigmaM "0"
         .TanDM "0.0"
         .TanDMFreq "0.0"
         .TanDMGiven "False"
         .TanDMModel "ConstTanD"
         .EnableUserConstTanDModelOrderMu "False"
         .ConstTanDModelOrderMu "1"
         .SetMagParametricConductivity "False"
         .DispModelEps  "Drude"
         .EpsInfinity "1.0"
         .DispCoeff1Eps "6.28*2042e12"
         .DispCoeff2Eps "71e12"
         .DispCoeff3Eps "0.0"
         .DispCoeff4Eps "0.0"
         .DispModelMu "None"
         .DispersiveFittingSchemeEps "Nth Order"
         .MaximalOrderNthModelFitEps "10"
         .ErrorLimitNthModelFitEps "0.1"
         .UseOnlyDataInSimFreqRangeNthModelEps "False"
         .DispersiveFittingSchemeMu "Nth Order"
         .MaximalOrderNthModelFitMu "10"
         .ErrorLimitNthModelFitMu "0.1"
         .UseOnlyDataInSimFreqRangeNthModelMu "False"
         .UseGeneralDispersionEps "False"
         .UseGeneralDispersionMu "False"
         .NLAnisotropy "False"
         .NLAStackingFactor "1"
         .NLADirectionX "1"
         .NLADirectionY "0"
         .NLADirectionZ "0"
         .Colour "1", "1", "0.501961" 
         .Wireframe "False" 
         .Reflection "False" 
         .Allowoutline "True" 
         .Transparentoutline "False" 
         .Transparency "0" 
         .Create
    End With
    '''
    modeler.add_to_history('define gold_drude', vba)
```

+ 定义有耗材料Si

```
def Si_lossy(modeler):
    vba = '''
    With Material
         .Reset
         .Name "Silicon (lossy)"
         .Folder ""
    .FrqType "all"
    .Type "Normal"
    .SetMaterialUnit "GHz", "mm"
    .Epsilon "11.9"
    .Mu "1.0"
    .Kappa "2.5e-004"
    .TanD "0.00"
    .TanDFreq "0.0"
    .TanDGiven "False"
    .TanDModel "ConstTanD"
    .KappaM "0.0"
    .TanDM "0.0"
    .TanDMFreq "0.0"
    .TanDMGiven "False"
    .TanDMModel "ConstKappa"
    .DispModelEps "None"
    .DispModelMu "None"
    .DispersiveFittingSchemeEps "General 1st"
    .DispersiveFittingSchemeMu "General 1st"
    .UseGeneralDispersionEps "False"
    .UseGeneralDispersionMu "False"
    .Rho "2330.0"
    .ThermalType "Normal"
    .ThermalConductivity "148"
    .SpecificHeat "700", "J/K/kg"
    .SetActiveMaterial "all"
    .MechanicsType "Isotropic"
    .YoungsModulus "112"
    .PoissonsRatio "0.28"
    .ThermalExpansionRate "5.1"
    .Colour "0.94", "0.82", "0.76"
    .Wireframe "False"
    .Transparency "0"
    .Create
    End With 
    '''
    modeler.add_to_history('define Silicon (lossy)', vba)
```

+ 定义材料SiO2

```
def SiO2(modeler):
    vba = '''
    With Material 
         .Reset 
         .Name "SiO2"
         .Folder ""
         .Rho "0.0"
         .ThermalType "Normal"
         .ThermalConductivity "0"
         .SpecificHeat "0", "J/K/kg"
         .DynamicViscosity "0"
         .Emissivity "0"
         .MetabolicRate "0.0"
         .VoxelConvection "0.0"
         .BloodFlow "0"
         .MechanicsType "Unused"
         .FrqType "all"
         .Type "Normal"
         .MaterialUnit "Frequency", "THz"
         .MaterialUnit "Geometry", "um"
         .MaterialUnit "Time", "ns"
         .MaterialUnit "Temperature", "Kelvin"
         .Epsilon "3.9"
         .Mu "1"
         .Sigma "0.0"
         .TanD "0.025"
         .TanDFreq "0.0"
         .TanDGiven "True"
         .TanDModel "ConstTanD"
         .EnableUserConstTanDModelOrderEps "False"
         .ConstTanDModelOrderEps "1"
         .SetElParametricConductivity "False"
         .ReferenceCoordSystem "Global"
         .CoordSystemType "Cartesian"
         .SigmaM "0"
         .TanDM "0.0"
         .TanDMFreq "0.0"
         .TanDMGiven "False"
         .TanDMModel "ConstTanD"
         .EnableUserConstTanDModelOrderMu "False"
         .ConstTanDModelOrderMu "1"
         .SetMagParametricConductivity "False"
         .DispModelEps "None"
         .DispModelMu "None"
         .DispersiveFittingSchemeEps "Nth Order"
         .MaximalOrderNthModelFitEps "10"
         .ErrorLimitNthModelFitEps "0.1"
         .UseOnlyDataInSimFreqRangeNthModelEps "False"
         .DispersiveFittingSchemeMu "Nth Order"
         .MaximalOrderNthModelFitMu "10"
         .ErrorLimitNthModelFitMu "0.1"
         .UseOnlyDataInSimFreqRangeNthModelMu "False"
         .UseGeneralDispersionEps "False"
         .UseGeneralDispersionMu "False"
         .NLAnisotropy "False"
         .NLAStackingFactor "1"
         .NLADirectionX "1"
         .NLADirectionY "0"
         .NLADirectionZ "0"
         .Colour "0.752941", "0.752941", "0.752941" 
         .Wireframe "False" 
         .Reflection "False" 
         .Allowoutline "True" 
         .Transparentoutline "False" 
         .Transparency "0" 
         .Create
    End With
    '''
    modeler.add_to_history('define SiO2', vba)

    # change color
    vba = '''
    With Material 
         .Name "SiO2"
         .Folder ""
         .Colour "0.752941", "0.752941", "0.752941" 
         .Wireframe "False" 
         .Reflection "False" 
         .Allowoutline "True" 
         .Transparentoutline "False" 
         .Transparency "0" 
         .ChangeColour 
    End With
    '''
    modeler.add_to_history('change color of SiO2', vba)
```

+ 定义brick

``` 
def brick(modeler, name, comp, material, Xrange, Yrange, Zrange):
    vba = '''
    With Brick
     .Reset 
     .Name "{0}" 
     .Component "{1}" 
     .Material "{2}" 
     .Xrange "{3[0]}", "{3[1]}" 
     .Yrange "{4[0]}", "{4[1]}" 
     .Zrange "{5[0]}", "{5[1]}" 
     .Create
    End With
    '''.format(name, comp, material, Xrange, Yrange, Zrange)
    modeler.add_to_history(f'define brick {name}', vba)
```

+ 定义旋转90°

```
def transform(modeler):
    vba = '''
    With Transform 
         .Reset 
         .Name "component1:line1" 
         .Origin "CommonCenter" 
         .Center "0", "0", "0" 
         .Angle "0", "0", "90" 
         .MultipleObjects "True" 
         .GroupObjects "False" 
         .Repetitions "1" 
         .MultipleSelection "False" 
         .Destination "" 
         .Material "" 
         .Transform "Shape", "Rotate" 
    End With 
    '''
    modeler.add_to_history('rotate line1 90', vba)
```

+ 定义布尔加操作

```
def boolean_add(modeler, name1, name2):
    vba = '''
    Solid.Add "component1:{0}", "component1:{1}" 
    '''.format(name1, name2)
    modeler.add_to_history(f'Add {name1} and {name2}', vba)
```

+ 定义floquet边界条件

```
def floquet_boundary(modeler):
    vba = '''
    With FloquetPort
         .Reset
         .SetDialogTheta "0" 
         .SetDialogPhi "0" 
         .SetPolarizationIndependentOfScanAnglePhi "0.0", "False"  
         .SetSortCode "+beta/pw" 
         .SetCustomizedListFlag "False" 
         .Port "Zmin" 
         .SetNumberOfModesConsidered "1" 
         .SetDistanceToReferencePlane "0.0" 
         .SetUseCircularPolarization "False" 
         .Port "Zmax" 
         .SetNumberOfModesConsidered "1" 
         .SetDistanceToReferencePlane "0.0" 
         .SetUseCircularPolarization "False" 
    End With
    '''
    modeler.add_to_history('set floquet boundary', vba)
```

+ 定义S参数提取

```
def get_S(mws):
    project = cst.results.ProjectFile(mws.filename(), allow_interactive=True)
    wl = np.array(project.get_3d().get_result_item("1D Results\S-Parameters\SZmax(1),Zmax(1)").get_xdata())
    s11 = np.array(project.get_3d().get_result_item("1D Results\S-Parameters\SZmax(1),Zmax(1)").get_ydata())

    s11_norm = np.abs(s11)
    s11_arg = np.angle(s11)
    result = np.transpose(np.stack([wl, s11_norm, s11_arg]))
    return result
```



Step3: 主函数：创建十字形的超表面结构，最终得到其吸收率的结果

![1672394218872](assets/1672394218872.png)

<center>创建的十字形超表面结构 </center>

```
tree = ModelTree('Model')
save_path = os.getcwd()  # save for cst files
save_name = 'demo.cst'
save_name = os.path.join(save_path, save_name)
print("Working file: {}".format(save_name))

start_time = time.time()
cstx = interface.DesignEnvironment()  # create new cst project and save it
mws = cstx.new_mws()
# mws.save(path=os.path.join(save_path, save_name))
mws.save(path=save_name)

mws.activate()
modeler = mws.modeler  # define the modeler to create the unit cell structure

# set base envs
setup_base(modeler)

# add materials
gold_drude(modeler)
SiO2(modeler)

# create pattern
# substrate
name = 'substrate'
comp = 'component1'
material = 'SiO2'
brick(modeler, name, comp, material, [-1, 1], [-1, 1], [-0.19/2, 0.19/2])
create_model(size=(2, 2, 0.19), position=(0, 0, 0), type='Box', color='grey', opacity=0.8)
tree.add_node('substrate')
# ground
name = 'ground'
comp = 'component1'
material = 'gold_drude'
brick(modeler, name, comp, material, [-1, 1], [-1, 1], [-0.19/2-0.1, -0.19/2])
create_model(size=(2, 2, 0.1), position=(0, 0, -0.145), type='Box', color='yellow', opacity=0.8)
tree.add_node('ground')

# # change color of gold_self
# ss.change_color(modeler, 'gold_self', [1, 1, 0.501961])

# line 1
name = 'line1'
comp = 'component1'
material = 'gold_drude'
brick(modeler, name, comp, material, [-0.5, 0.5], [-0.1, 0.1], [0.19/2, 0.19/2+0.06])
transform(modeler)
create_model(size=(1, 0.2, 0.06), position=(0, 0, 0.125), type='Box', color='yellow',opacity=0.8)
tree.add_node('line1')

# boolean add
name1 = 'line1'
name2 = 'line1_1'
boolean_add(modeler, name1, name2)
create_model(size=(0.2, 1, 0.06), position=(0, 0, 0.125), type='Box', color='yellow',opacity=0.8)
tree.add_node('line2')
floquet_boundary(modeler)
# run
modeler.run_solver()


# get result
# result = ss.get_R_A(mws)
result = get_S(mws)

mws.close()
cstx.close()

# end and save results
end_time = time.time()
# print("time elapse: {:.2f} s".format(end_time - start_time))
np.savetxt('simulation result.txt', result)

# plot Absorbance and Reflectance
wl = 3*(10**2)/result[:, 0]
s11 = result[:, 1]
# A = result[:, 2]
A = 1 - s11 ** 2
plt.figure()
plt.plot(wl, A, label='Absorbance')
plt.legend()
plt.xlabel('wavelength(um)')
plt.savefig('Absorbance')
device.save_fig('Absorbance', file_path="Absorbance.png")
```

