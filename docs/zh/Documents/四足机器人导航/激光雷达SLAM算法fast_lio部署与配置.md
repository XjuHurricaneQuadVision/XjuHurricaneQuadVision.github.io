> 作者：新疆大学 计算机科学与技术 黄耀增

> 网上很多fast_io都是基于ROS的而不是ROS2，所以记得甄别。

这边我找了一个[fast_lio2的开源地址](https://github.com/Ericsii/FAST_LIO_ROS2)在github

下载完后的应该是一个名叫FAST_LIO_ROS2_ros2之类的名字

我建议使用vscode全局将此类名字更改为fast_lio,这是真理，不然就有可能g

剩下的就是正常编译以及添加source等等，注意开两个终端

分别运行以下代码：

```PowerShell
ros2 launch livox_ros_driver2 msg_MID360_launch.py

ros2 launch fast_lio mapping_mid360.launch.py


# 应该知道哪个运行哪个的吧
```

> 不过开两个终端太烦了，我们来整合一下`msg_MID360_launch.py`和`mapping_mid360.launch.py`这两个launch文件。

首先在fast_lio功能包的launch文件夹里面建一个启动文件，名字随意，然后输入如下代码：

```Python
import os.path
import os

from ament_index_python.packages import get_package_share_directory

import launch
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument
from launch.substitutions import LaunchConfiguration, PathJoinSubstitution
from launch.conditions import IfCondition

from launch_ros.actions import Node



xfer_format   = 1    # 0-使用 PointCloud2 格式（PointXYZRTL）, 1-自定义点云格式
multi_topic   = 0    # 0-所有 LiDARs 共享同一话题, 1-每个 LiDAR 使用独立话题
data_src      = 0    # 0-数据来源于 lidar, 其他数字属于无效数据源
publish_freq  = 10.0 # 发布频率, 5.0, 10.0, 20.0, 50.0, etc.
output_type   = 0
frame_id      = 'livox_frame'  # 坐标框架ID

# 指定 LiDAR 数据文件的路径
lvx_file_path = '/home/hyz/project/ws_livox/src/Indoor_sampledata.lvx2'

cmdline_bd_code = 'livox0000000001'

 # 完整的用户配置文件路径
user_config_path = '/home/hyz/project/ws_livox/src/livox_ros_driver2/config/MID360_config.json'


livox_ros2_params = [
    {"xfer_format": xfer_format},
    {"multi_topic": multi_topic},
    {"data_src": data_src},
    {"publish_freq": publish_freq},
    {"output_data_type": output_type},
    {"frame_id": frame_id},
    {"lvx_file_path": lvx_file_path},
    {"user_config_path": user_config_path},
    {"cmdline_input_bd_code": cmdline_bd_code}
]



def generate_launch_description():

    livox_driver = Node(
        package='livox_ros_driver2',
        executable='livox_ros_driver2_node',
        name='livox_lidar_publisher',
        output='screen',
        parameters=livox_ros2_params
        )
    


    package_path = get_package_share_directory('fast_lio')
    default_config_path = os.path.join(package_path, 'config')
    default_rviz_config_path = os.path.join(
        package_path, 'rviz', 'fastlio.rviz')

    use_sim_time = LaunchConfiguration('use_sim_time')
    config_path = LaunchConfiguration('config_path')
    config_file = LaunchConfiguration('config_file')
    rviz_use = LaunchConfiguration('rviz')
    rviz_cfg = LaunchConfiguration('rviz_cfg')

    declare_use_sim_time_cmd = DeclareLaunchArgument(
        'use_sim_time', default_value='false',
        description='Use simulation (Gazebo) clock if true'
    )
    declare_config_path_cmd = DeclareLaunchArgument(
        'config_path', default_value=default_config_path,
        description='Yaml config file path'
    )
    decalre_config_file_cmd = DeclareLaunchArgument(
        'config_file', default_value='mid360.yaml',
        description='Config file'
    )
    declare_rviz_cmd = DeclareLaunchArgument(
        'rviz', default_value='true',
        description='Use RViz to monitor results'
    )
    declare_rviz_config_path_cmd = DeclareLaunchArgument(
        'rviz_cfg', default_value=default_rviz_config_path,
        description='RViz config file path'
    )

    fast_lio_node = Node(
        package='fast_lio',
        executable='fastlio_mapping',
        parameters=[PathJoinSubstitution([config_path, config_file]),
                    {'use_sim_time': use_sim_time}],
        output='screen'
    )
    rviz_node = Node(
        package='rviz2',
        executable='rviz2',
        arguments=['-d', rviz_cfg],
        condition=IfCondition(rviz_use)
    )

    ld = LaunchDescription()
    ld.add_action(declare_use_sim_time_cmd)
    ld.add_action(declare_config_path_cmd)
    ld.add_action(decalre_config_file_cmd)
    ld.add_action(declare_rviz_cmd)
    ld.add_action(declare_rviz_config_path_cmd)

    ld.add_action(livox_driver)
    ld.add_action(fast_lio_node)
    ld.add_action(rviz_node)

    return ld
```

> 其中`lvx_file_path`目前我还没花时间研究有什么实质性的用处，大概是类似于数据集训练出的模型的东西，帮助得到更好的建图效果。

> 其中`user_config_path`参数需要`/ws_livox/src/livox_ros_driver2/config/MID360_config.json`的绝对位置，你们自行在前端补全。

> 保存pcd路径在这个包里面怎么改我还在研究，敬请期待！

