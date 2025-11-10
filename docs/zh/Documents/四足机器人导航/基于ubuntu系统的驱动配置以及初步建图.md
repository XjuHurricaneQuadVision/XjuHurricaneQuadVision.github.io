> 作者：新疆大学 计算机科学与技术 黄耀增

Linux系统 : ubuntu 22.04

ROS2 : humble

参考教程：[ROS2+mid360建图教程（1）](https://blog.csdn.net/Sixfourjie/article/details/138243730)

[ROS2+mid360建图教程（2）](https://blog.csdn.net/Sixfourjie/article/details/138278812)

[编译livox ros driver2（ROS2、livox、rviz、ubuntu22.04）](https://blog.csdn.net/qq_29912325/article/details/130269367?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522169734904416800182711632%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=169734904416800182711632&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-2-130269367-null-null.142%5Ev96%5Epc_search_result_base9&utm_term=livox_sdk2&spm=1018.2226.3001.4187)

也算是实验室第一次搞激光雷达，网上教程多但是由于也是实验室第一次部署ROS2,或多或少有些迷茫，不过加油，相信马术才是最吊的！

# 一、依赖配置

> 在我们运行之前，还需要用到如下几个依赖：

```Shell
pip install catkin_pkg
pip install empy==3.3.4
pip install numpy
pip install lark
sudo apt-get install ros-humble-pcl-ros
sudo apt install ros-humble-tf-transformations
sudo pip3 install transforms3d
```

> `rclpy`

```Go
import rclpy的时候出现 "libstdc++.so.6: version `GLIBCXX_3.4.30' not found"
可以在base环境中试一下 conda install -c conda-forge gcc=12.1.0
```



# 二、驱动配置

## 1.安装Livox-SDK2

> [Livox-SDK2地址](https://github.com/Livox-SDK/Livox-SDK2)在github

```PowerShell
git clone https://github.com/Livox-SDK/Livox-SDK2.git
cd ./Livox-SDK2/
mkdir build
cd build
cmake .. && make
sudo make install
```

> 生成的共享库和静态库安装到“/usr/local/lib”目录下。头文件安装到“/usr/local/include”目录下

## 2.安装livox_ros_driver2

> [livox_ros_driver2地址](https://github.com/Livox-SDK/livox_ros_driver2)在github

```PowerShell
git clone https://github.com/Livox-SDK/livox_ros_driver2.git ws_livox/src/livox_ros_driver2

#终端进入到/src/livox_ros_driver2目录下
source /opt/ros/humble/setup.sh
./build.sh humble


#如果4.5行教程不行的话建议用ROS2最常用的编译方法,
#终端进入到/ws_livox目录下
colcon build  #如果里面有别的功能包，学过ROS2的应该知道在后面加--packages-select来指定编译
source install/setup.bash
```

> 之后再把config文件夹中的MID360_config.json文件修改了：

```PowerShell
{
  "lidar_summary_info" : {
    "lidar_type": 8
  },
  "MID360": {
    "lidar_net_info" : {
      "cmd_data_port": 56100,
      "push_msg_port": 56200,
      "point_data_port": 56300,
      "imu_data_port": 56400,
      "log_data_port": 56500
    },
    "host_net_info" : {
      "cmd_data_ip" : "192.168.1.50",    # 这四个我标记的地方改成和设置那边IPv4一样的ip就行
      "cmd_data_port": 56101,
      "push_msg_ip": "192.168.1.50",     # 你完全可以复制我这个，完全可用
      "push_msg_port": 56201,
      "point_data_ip": "192.168.1.50",   # 对的就是这四个
      "point_data_port": 56301,
      "imu_data_ip" : "192.168.1.50",    # 四个截至到这里
      "imu_data_port": 56401,
      "log_data_ip" : "",
      "log_data_port": 56501
    }
  },
  "lidar_configs" : [
    {
      "ip" : "192.168.1.146",            # 这个不一样了,ip最后改成1+雷达序列码后两位(雷达本体旁边贴纸）
      "pcl_data_type" : 1,
      "pattern_mode" : 0,
      "extrinsic_parameter" : {
        "roll": 0.0,
        "pitch": 0.0,
        "yaw": 0.0,
        "x": 0,
        "y": 0,
        "z": 0
      }
    }
  ]
}

#！！！！！！！！！！记住除了我标注的地方其他地方不用动！！！！！！！！！！
```



# 三、主机配置

进入主机设置-网络，然后在有线处添加，名字随便你取，我们一般取mid-360，然后点击IPv4，改为手动，在底下添加如下所示：

![mid-360_1.png](/img/Documents/mid-360_1.png)

最后一切设置完毕后就点击右上角的应用即可。

# 四、功能包运行

> 终端进入到/src/livox_ros_driver2目录下，将激光雷达与电脑连接，输入以下指令就可以看到在Rviz2上生成的点云图了。

```PowerShell
ros2 launch livox_ros_driver2 rviz_MID360_launch.py
```

