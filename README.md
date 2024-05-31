# 方舟无限Diffusion Policy部署框架

## 源文件编译

如果没有修改源文件，编译只需要执行一次：

```shell
conda deactivate
[follow1] catkin_make
[follow2] catkin_make
```

新建一个终端执行:

```shell
roscore
```

## 打开can通信

```shell
./can.sh
```

## 启动机械臂

根据需求打开指定的机械臂文件夹：

```shell
# 打开一个新的终端
[follow2] source devel/setup.bash && roslaunch arm_control arx5.launch
```

## 启动摄像机

新建终端执行：

```shell
[follow1] source devel/setup.bash && roslaunch arm_control camera.launch
```

## 真机部署

新建一个终端执行插值程序：

```shell
[follow1] source devel/setup.bash && rosrun arm_control linear_interpolation.py
```

另外建一个终端执行推理程序：

```shell
[follow1] source devel/setup.bash
[KUKA-Controller] python eval_real_ros.py --input_path /path/to/weight
```