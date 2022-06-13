为什么使用cyberrt与ros在自动驾驶的缺陷
#######################################################
ros的不足：大数据量传输性能瓶颈；单中心的网络存在单点风险；数据格式缺乏后相兼容
1帧lidar数据大小是7m左右，一个cemral是5m左右；大数据瓶颈导致了时延，而且时延太长会丢弃数据，这在自动驾驶中非常危险；不能满足实际工程；
原生ros每次把一个节点的数据传输给另一个节点要经过四次copy，这是一对一情况，一对多就4*n
cyberRT则采用共享内存的方式来存储数据，这样每次只有两次copy
nvidia提供的drive px2板卡有两个系统，其中一个进行冗余备份；
cyberRT去掉了中心拓扑，建立一个点对点的复杂的网络拓扑；使用rtps 

############################################################
按照步骤来
要配置nvidia docker toolkit
cd ~/apollo
bash docker/scripts/dev_start.sh
bash docker/scripts/dev_into.sh
bash scripts/bootstrap.sh

modules/calibration/data 用来保存车辆数据
modules/map/data 用来保存地图数据
在容器内可以直接使用apollo功能，无需执行构建过程
record是apollo记录数据的一种数据格式，以.record为后缀的文件就是我们说的record数据包；
cyber_recorder play -f name --loop播放apollo演示包；
username@computername:~$: source /apollo/cyber/setup.bash配置cyberRT；
###################################################
使用cyberRecorder播放记录
cyber_recorder play -f sensor_rgb.record -loop
打开新的终端，启动cyber_monitor来查看channel数据
这次进入docker只需要执行docker/scripts/dev_into.sh即可；
cyber_monitor
#####################################################3
dreamview启动所需要的模块
打开transform
cyber_launch start /apollo/modules/transform/launch/static_transform.launch
打开image decompression
cyber_launch start modules/drivers/tools/image_decompress/launch/image_decompress.launch
打开红绿灯检测
cyber_launch start /apollo/modules/perception/production/launch/perception_trafficlight.launch
打开视觉障碍物检测模块
mainboard -d modules/perception/production/dag/dag_streaming_perception_camera.dag
查看gpu占用情况
  watch -n 0.1 nvidia-smi
  单独启动车道线检测模块
  mainboard -d ./modules/perception/production/dag/dag_streaming_perception_lane.dag
  #######################感知部分代码目录结构###############################3
  apollo/modules/perception/camera: 视觉算法核心模块
  apollo/modules/perception/lidar: 激光雷达算法核心模块
  apollo/modules/perception/radar: 毫米波雷达算法核心模块
  apollo/modules/perception/fusion: 多传感器融合算法核心模块

卡尔曼滤波器来进行目标跟踪，缺陷是实时性较差，复杂度较高

  
