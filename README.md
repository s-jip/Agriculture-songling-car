# Agriculture-songling-car
一，松灵Hunter se底盘
序号
介绍
路径
1
松灵hunter se离线网页
HUNTER SE 用户手册.mhtml 
2
在线手册
HUNTER SE 用户手册 · AgileX知识库
3
sdk (v2)
https://github.com/agilexrobotics/agx_sdk
4
sdk (v1)
https://github.com/agilexrobotics/ugv_sdk
5
ros2_pack
https://github.com/agilexrobotics/hunter_ros2
6
开源文档
agilexrobotics - Overview
7
下载gs_usb内核
https://forums.developer.nvidia.com/t/does-jetpack-6-0-not-support-gs-usb-usb-to-can/328702/17
底盘官方轮径（wheel_radius）：0.1375m

# 1. 下載自動化編譯腳本
wget -q https://github.com/lucianovk/jetson-gs_usb-kernel-builder/raw/main/jetson-gs_usb-kernel-builder.sh

# 2. 給腳本賦予執行權限
chmod +x jetson-gs_usb-kernel-builder.sh

# 3. 用超級管理員權限運行它（這會自動幫你下載內核頭文件並編譯出 gs_usb 驅動）
sudo ./jetson-gs_usb-kernel-builder.sh

# 4. 查看gs_usb内核是否正常加载（不报错即可）
sudo modprobe gs_usb

sudo apt-get update
sudo apt-get install -y libasio-dev
# 1. 查看can通信接口情况
cd ~/songling_ws/src/ugv_sdk/scripts
bash find_all_can_port.sh

# 2. 根据查询的can口情况，分别激活Orin（MTTCAN）和底盘can口
cd ~/songling_ws/
sudo ip link set can0 up type can bitrate 500000
sudo ip link set can1 up type can bitrate 500000

# 若显示设配 Device or resource busy，则先进行以下这步
sudo ip link set can0 down
sudo ip link set can1 down
# 查看通讯情况（查看底盘的can口）
candump can0

# 启动底盘节点
source install/setup.bash 
ros2 launch hunter_base hunter_base.launch.py
#底盘can口修改在/home/nvidia/songling_ws/src/hunter_ros2/hunter_base/launch/hunter_base.launch.py

发布话题
/cmd_vel
/hunter_status
/odom
/parameter_events
/rosout
/tf

# 启动键盘
cd ~/songling_ws
source /opt/ros/humble/setup.bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
二，WHEELTEC G90双天线定位模块
序号
介绍
路径
1
产品资料
https://pan.baidu.com/s/1DylwAarob5WvgbDtZaI8xg
2
产品Agent助手
https://agent.dingtalk.com/copilot?code=tWeS4Xqs2Y
#步骤一：配置串口二输出
gpgga com2 0.1 
gprmc com2 0.1
saveconfig 
#按【Enter】键完成指令发送

#步骤二：配置 Type-C 主串口输出数据
GPGGA 0.05 
GPRMC 0.05 
GPHDT 0.1 
GPVTG 0.2 
GPGSV 0.2 
GPGST 0.2 
AGRICA 1 
PVTSLNA 1 
BESTNAVA 1 
SAVECONFIG 
#按【Enter】键完成指令发送
# 安装 Python 依赖
pip3 install transforms3d serial pyserial um982-driver

# 安装 ROS2 依赖（根据你的 ROS2 版本，这里是 humble）
sudo apt update
sudo apt install ros-humble-tf-transformations ros-humble-gps-umd

# 或者使用 rosdep 一键安装（推荐）
cd ~/ros2_ws   # 进入你的工作空间
rosdep install --from-paths src --ignore-src -r -y

# 安装 rosdepc
sudo pip install rosdepc

# 然后初始化并更新
sudo rosdep init
rosdep update
# 1. 查看所有可能出现的串口设备节点
ls -l /dev/ttyUSB* /dev/ttyACM* /dev/serial/by-id/* 2>/dev/null

# 2.将当前用户加入 dialout 组（确保有读写权限）执行后 注销当前终端会话并重新登录（或者重启 Orin）
sudo usermod -a -G dialout $USER

# 3.使用正确的波特率读取
sudo stty -F /dev/ttyACM0 115200
cat /dev/ttyACM0
cat /dev/wheeltec_gnss
# 用 udevadm 查看详细信息：
udevadm info -a -n /dev/ttyACM0 | grep -i serial

# 更改 wheeltec_gnss.sh文件参数
ATTRS{serial}=="0005" ----> ATTRS{serial}=="对应信息"

在这个目录下/home/nvidia/songling_ws/src/wheeltec_gps_driver/src/
文件赋权：sudo chmod 777 wheeltec_gnss.sh 
执行脚本：sudo sh wheeltec_gnss.sh
#启动RTK定位
cd ~/songling_ws
source install/setup.bash
ros2 launch wheeltec_gps_driver wheeltec_dual_rtk_driver_nmea.launch.py port:=/dev/ttyACM0

发布的话题
/gnss/gpgga
/gps/fix
/gps/nmea/vel
/gps/pose
/gps/vel
/heading
/heading_deg
/nmea_sentence
/parameter_events
/rosout
/time_reference

# 查看节点信息
source ~/songling_ws/install/setup.bash
ros2 topic echo /gps/fix
三 ，宇树L2激光雷达
1
宇树L2相关说明
https://www.unitree.com/cn/download/L2
2
建图Point-LIO算法(ROS2版)
https://github.com/dfloreaa/point_lio_ros2

# 1. 更新系统源
sudo apt update

# 2. 安装 rosdep 工具（如果新机还没装的话）
sudo apt install python3-rosdep -y

# 3. 初始化并更新 rosdep（如果提示已初始化，直接执行 update 即可）
sudo rosdep init
rosdep update

# 4. 进入你的工作空间
cd ~/songling_ws

# 5. 神奇指令：让 rosdep 自动读取 src 目录下所有包的 package.xml 并安装缺失依赖
rosdep install --from-paths src --ignore-src -r -y

# 6. 确保核心底层库都存在
sudo apt install ros-humble-pcl-ros ros-humble-pcl-conversions libpcl-dev -y
cd ~/songling_ws
source install/setup.bash
ros2 launch unitree_lidar_ros2 launch.py

发布话题
/clicked_point
/goal_pose
/initialpose
/parameter_events
/rosout
/tf
/tf_static
/unilidar/cloud
/unilidar/imu
cd ~/songling_ws
source install/setup.bash 
ros2 launch songling_navigation mapping_l2.launch.yaml
cd ~/songling_ws/src/songling_navigation/PCD
pcl_viewer final_map.pcd # 目录要在保存地图的文件夹中
# 接收、压缩地图节点启动（终端1）
cd /home/nvidia/songling_ws
source install/setup.bash
ros2 launch pcd2pgm pcd2pgm_launch.py

# 发送地图（终端2）进行两次  --第一次会出现报错然后rviz2中出现压缩好的2D图  --第二次保存
cd /home/nvidia/songling_ws
source install/setup.bash
ros2 run nav2_map_server map_saver_cli -t /map -f /home/nvidia/songling_ws/src/songling_navigation/PCD/final_map --ros-args -p save_map_timeout:=10.0

五，Jetson Orin Nano super mini（8G-256G）
Mini Jetson Orin NX 及Orin Nano 百度网盘链接：https://pan.baidu.com/s/1LWNhd1yMH54ktkpvUJwVrg 提取码：iftk
序号
介绍
远程密码
远程桌面
ID
Key
1
松灵车远程信息（使用Rustdesk连接）
Songling307

517989452
116.62.225.194
gO92JbniudgxDPVbcZ6ph0ORIqshO1c+v+MF3yaqVFo=

sudo apt-mark hold 'nvidia-l4t-*'  
六.乐视三合一深度体感摄像头三维rgbd相机ros机器人astra pro（摄像头已更换需重新做）
开发资料：
请至社区下载(驱动、SDK、论坛)：https://developer.orbbec.com.cn/forum_plate_module_details.html?id=13476
暂时无法在飞书文档外展示此内容

---
sudo apt install -y libgflags-dev nlohmann-json3-dev \
ros-humble-image-transport ros-humble-image-publisher \
ros-humble-magic-enum libuvc-dev
cd ~/songling_ws/src/ros2_astra_camera/astra_camera/scripts
sudo bash install.sh
sudo udevadm control --reload-rules && sudo udevadm trigger
cd ~/songling_ws
colcon build --packages-select astra_camera_msgs
colcon build --packages-select astra_camera
# 启动节点
cd ~/songling_ws
source install/setup.bash
ros2 launch astra_camera astra.launch.py

# 发布话题
/camera/color/camera_info  #彩色相机内参（焦距、主点、畸变系数等）
/camera/color/image_raw    #彩色图像（RGB，分辨率 640x480 或 1920x1080）
/camera/depth/camera_info
/camera/depth/color/points #彩色 3D 点云（包含 RGB 颜色信息）
/camera/depth/image_raw
/camera/depth/points       #3D 点云（PointCloud2，仅深度数据）
/camera/extrinsic/depth_to_color  #深度到彩色相机的旋转/平移矩阵（外参）
/camera/ir/camera_info 
/camera/ir/image_raw
/parameter_events
/rosout

Astra Pro 相机
    ├── 彩色流 → /camera/color/image_raw → YOLO 输入
    │                    ├── /camera/color/camera_info → 3D 投影
    │                    └── /camera/depth/color/points → 彩色点云
    ├── 深度流 → /camera/depth/image_raw → detect_3d_node（测距）
    │                    ├── /camera/depth/camera_info → 3D 投影
    │                    └── /camera/depth/points → 深度点云
    ├── 红外流 → /camera/ir/image_raw → 低光照辅助
    └── 外参 → /camera/extrinsic/depth_to_color → 深度-彩色对齐

# 查看画面
rviz2
cd ~/songling_ws
source install/setup.bash
ros2 launch yolo_bringup songling_yolo.launch.py

#发布话题
/yolo/dbg_image
/yolo/debug_node/transition_event
/yolo/detect_3d_node/transition_event
/yolo/detections
/yolo/detections_3d      # 3D 目标检测结果（类别 + 2D框 + 3D位置/尺寸）
/yolo/dgb_bb_markers     # 3D边界框的 RViz 可视化标记（MarkerArray）
/yolo/dgb_kp_markers     # 关键点标记（仅姿态估计模型时有效）
/yolo/yolo_node/transition_event
七. 多节点启动
colcon build --packages-select songling_navigation
sudo pkill -9 -f hunter
# 室外GPS导航启动节点
cd ~/songling_ws
source install/setup.bash
ros2 launch songling_navigation gps_navigation.launch.py

# 发布多个GPS坐标 自动导航
cd ~/songling_ws
source install/setup.bash
ros2 run songling_navigation gps_waypoint_sender2.py

八.ros2常用命令总结
ros2 pkg list                                  查看功能包列表

colcon build –packages-select package_name     编译指定功能包
ros2 node list                                 查看所有运行节点

ros2 node info node_name                       查看某一个节点信息
ros2 topic list                                查看系统内活跃topic

ros2 topic type topic_name                     查看topic的消息类型

ros2 topic echo topic_name                     显示topic内容

ros2 topic hz topic name                       查看发布数据的帧率

ros2 topic pub topic_name msg_type 'args'      向特定topic发送数据
args是实际数据，需要以YAML格式输入
ros2 topic pub --once /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"
--once是可选参数，表示发布一个消息然后退出
ros2 topic pub --rate 10 /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0,
y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"
ros2 param list                查看节点的子命名空间和参数

ros2 param get node_name param_name         获取当前参数的值

ros2 param set node_name param_name value   设置参数的值 如 ros2 param set /turtlesim background_r 150
ros2 service list                # 加-t可显示服务消息类型 查看服务列表

ros2 service type service_name   查看服务的数据类型
 
ros2 service find type_name      查找使用指定数据类型的服务

ros2 service call <service_name> <service_type> <args>  通过命令行调用服务
ros2 action list        # 加 -t 可显示动作消息类型 列出域内可用的动作列表

ros2 action info action_name   查看指定action的信息

ros2 action send_goal <action_name> <type_name> <goal_data> <--feedback>  模拟客户端手动发起动作请求

