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
