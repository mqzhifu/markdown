# unity学习(概览)

## CG建模：

1. blender
2. 3DMax
3. maya
4. Cinema4D
5. ZBrush
6. ......

## 安装

unity.cn 登陆，下载hub，安装

打开hub,授权licence ，安装edtor，最好是选择lts\(Long Term Support \)

选择模块：

1. vs\(3.57G \)
2. andiord:
    1. sdk & ndk tools 4.21g
    2. OPEN JDK 157m
3. mac builder\(il2cpp\) 1.68G
4. documentation\(634\)
5. webGL build 1.74G
6. unity自带的IDE，大概3.7G左右

> 感觉是真的有点离谱... 这要是：安卓\+MAC\+WEBGL\+UNITY\-IDE 都安装上，15个GB了....安卓 WEBGL 先不安装了，正常：VS\+MAC\+DOC
> 
> 
> 安装中，果然遇到某些包\(unity editor\)失败的情况，无奈又得开VPN

## IDE

窗口：

1. tool bar 工具栏
2. hierarchy 层次结构
3. scene 场景
4. inspector 检查器/属性
5. project 项目
6. console 控制台

视图按钮\-删除网格

视图按钮\-删除天空

右侧窗口：Inspector

这个窗口日常使用应该最频繁，主要它是挂着各种组件

#### 坐标系

红色：X轴，东西

绿色：Y轴，上下

蓝色：Z轴，南北

local:以当前物体的形状做为坐标系

> 物体的形状可能是向左向上的，不规则的，移动的时候得按照自己的坐标系移动

global:以整个世界做为坐标系

> 世界只有一个，或者说一个场景就一个世界坐标系，就是最中心的位置，不可变，也没有参照物

pivot:轴心模式

> 物体的 XYZ 就在物体的中心位置

center:几何中心模式

> 多物体的 XYZ 就在多物体的中心位置

好像是 pivot 使用更多一些

#### 移动一个对象/物体：

拖动一个物体上的 XYZ 箭头\(向量指针\)

> 也可以点击该物体后，有3个面，拖动也可以移动物体

注：搬动的时候是 local 还是 global

#### 视角查看

旋转：alt \+ 鼠标左键\(left mouse button\)

> 以当前点做为中心，可以上下左右查看当前场景，注：是以一个点为中心，不能换视角

缩放/放大：用鼠标滚轮

拖动模式：鼠标中键，或 W 快捷建

将一个物体，F键，置于世界中心，才可以以其为中心位置 旋转

透视视图 perspective :近大远小，与距离有关系

正交视图 orthographic 又称等距视图 isometric，与距离无关。 对齐使用

#### 物体/对象

> 尺寸单位是一个单位：约1米.平面是10X10 . 所有对象的内部是不做渲染的\(背面是不渲染的\)

cube 正方

sphere 球体

capsule 胶囊

cylinder 圆柱

plane 平面\(没有厚度，从上可见，从下看是透明。正面是渲染的，背面是不渲染的

#### 物体旋转

选中一个物体：快捷键 E ，也可以点击菜单图标。之后拖动 XYZ

旋转：顺时针是负值，逆时针是正值。

按住ctrl:每次旋转以15为步长，也可以DIY

#### 物体绽放

选中一个物体：快捷键 R ，也可以点击菜单图标。之后拖动 XYZ

#### 网格模型

所有的模型，包括：UNITY 里的 cube、面、sphere 等，都是由一个个面组合而成，内部为空。每个面即：顶点坐标、法向等数据。GPU依此渲染，最终是一个完整的模型，在空间中的形状了。

> 对象内部空是因为：用户看不见的面，如果渲染是浪费
> 
> 
> 面数越多，效果越好\(光滑、圆润\)，当然GPU的负载越高

#### 材质 material

1. 颜色
2. 金属
3. 光滑/粗糙
4. 透明
5. 突起
6. .....

> 它定义了一个物体的密度\(硬度\)、反光等，扩展名：.mat

#### 纹理 texture \(贴图，PSD\)

建模软件导出格式：FBX\(mesh\+material\)

创建公共文件夹：model,里面放从外部导入的模型

另外还得再导入：贴图文件

场景文件：.unity \(保存一个场景内的所有信息，如：实体、实体的属性等等吧\)

unity官方资源：unity asset sotre

pivot:轴心点，这个轴心点不一定非得在一个物体的中心位置 ，可以在物体的任意位置

子节点的position 是以父节点的 基准点计算的

empty object\(只有 transform\)

1. 把零散的节点，放到子节点下面，统一管理
2. 标记空间的一个位置

global:只能有6个方向，上下 东西南北

local:如果一个物体不属于该方向，如：一个小车是向东北方向，那就用本地坐标系，自行设置，拖动

## 组件

先看一下几个常见/基础的组件：

1. mesh filter：加载 点线面的数据
2. mesh render：渲染 点线面的数据，在显示器上显示一个mesh

> 1和2加一起，也可以间接理解为：一个物体的显示\(渲染\)过程

1. light:光照
2. transform:位置 旋转角度 缩放比例\(不能被删除\)
3. camera:摄像。最终玩家进入游戏的角度就是它。\(game窗口就是它\)

> 调整到场景的观察角度 选中 camera object \-\> gameObject \-\> align with view

1. script:一个物体上绑定的c\#脚本

## 调试/预览

点击 右三角的图标，即运行了程序，看结果在另外一个窗口。

再点一下 右三角图标即停止该脚本

运行中调参数值的时候，如果想保存得右击组件

## 脚本

创建一个目录：asset/script

随便创建一个文件:x.cs

把该脚本拖动到一个物体上，即：物体多了一个组件

```
using System.Diagnostics;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using static UnityEditor.PlayerSettings;
using System.Numerics;

public class Move : MonoBehaviour
{
    private GameObject target;//向某个对象看齐
    public int coordinateType = 1;//1 相对坐标系 2 世界坐标系
    public int speed = 1;//每秒移动速度

    // Start is called before the first frame update
    void Start()
    {
        UnityEngine.Debug.Log("start:");
        //GameObject obj = this.gameObject;
        //target = GameObject.Find("imtarget");
        

        string name = this.name;
        UnityEngine.Debug.Log("obj name:"+name);
        Vector3 pot =  this.transform.position;
        UnityEngine.Debug.Log("obj position:" + pot);

    }

    // Update is called once per frame
    void Update()
    {
        this.transform.LookAt(target.transform);
        //float speed = 1f;//平均移动速度

        //if (coordinateType == 1)
        //{
        //    this.transform.Translate(0, 0, Time.deltaTime * speed, Space.Self);
        //}
        //else
        //{
        //    this.transform.Translate(0, 0, Time.deltaTime * speed, Space.World);
        //}

        
    }

    //void moveBySelf()
    //{

    //    Vector3 Pos = this.transform.localPosition;//获取当前物体的位置
    //    pos.x += Time.deltaTime * speed;//沿着 X 轴，按照一定速度：移动一次
    //    this.transform.localPosition = pos;
    //}
}

```

NewBehaviourScript:类名，这个得和文件名一致，不然报错

MonoBehaviour: unity 所有类的基类，所有脚本都得继承它。后面再讲吧

公共函数：

1. awake:有唤醒即执行
2. onEnable
3. start:物体激活即执行，只执行一次
4. update:每一帧的变化都会回调此函数
5. onDisable
6. onDestroy

> 这就是安卓的 activity，或者说：前端的开发的 callback 模式

几个常用的类：

1. UnityEngine: unity 引擎类

> UnityEngine.Debug.Log

1. GameObject: 一个对象
2. Transform: 物体的位置信息组件

> transform.position
> 
> 
> transform.localPosition

1. time:时间相关的

> Time.deltaTime;
> 
> 
> Time.time

1. this:代表当前脚本挂载的物体对象

> this.gameObject
> 
> 
> this.transform.localPosition

设置 FPS 固定帧率：

Application.targetRate = 60

> 不过 unity 也不可能完全精准做到每秒就是60帧

物体移动

物体父子关系

物体旋转

主控脚本

组件间的调用

查找/获取 一个物体：

GameObject.Find\("path"\)

## 总结

贴图者 \-\> PS 画对象\(二维模型\+二维贴图\)

建模者 \-\> 创建模型、贴皮、选择材质，合并子模型，创建pivot

前端unity开发者

1. 导入asset资源，将模型加载到unity场景中

2. transform

3. mesh filter \+ mesh render

4. light / 重力 / audio source /

> 场景及模型都在UNITY中，初始化完成

5. 开始添加动态\(c\#\)脚本
