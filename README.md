如何在Hisi3559上使用ROS Kinetic

1. 拉取hisi_kinetic docker镜像
   ```
   docker pull uupks/hi3559:hisi_kinetic
   ```
2. 创建catkin工作空间
   ```
   mkdir catkin_arm_ws
   cd catkin_arm_ws
   catkin init
   catkin config --merge-devel
   catkin config --merge-install
   mkdir src
   ```
3. 拉取hisi_demo
   ```
   cd catkin_arm_ws/src
   git clone https://github.com/uupks/hisi_ros_demo.git
   ```
4. 运行hisi_kinetic容器
   ```
   docker run -it --rm -v {your absolute path to catkin_arm_ws}:/catkin_arm_ws hisi_kinetic
   ```
5. 将容器中相关文件拷贝或者通过nfs挂载到Hi3559开发板
   ```
   docker cp {container id}:/hisi-linux .
   docker cp {container id}:/ros-arm .
   ```
6. 在容器中进行编译
   ```
   cd /catkin_arm_ws
   catkin build --force-cmake -DCATKIN_ENABLE_TESTING=OFF -DROSCONSOLE_BACKEND=print --cmake-args -DCMAKE_TOOLCHAIN_FILE=/hisi_kinetic_toolchain.cmake
   ```
7. 编译成功后在主机上将 `catkin_arm_ws` 目录挂载到开发板上
8. 进入开发板设置一些环境变量
   ```
   # 设置运行时lib路径
   export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/hisi-linux/lib
   # 设置hosts，将 192.168.50.241 改为你自己的设置
   vi /etc/hosts
   127.0.0.1 localhost
   192.168.50.241 hisi
   # 设置ros hostname
   export ROS_HOSTNAME=hisi
   # ROS环境变量设置
   source /ros-arm/setup.sh
   ```
9. 运行hisi_demo
   ```
   cd /catkin_arm_ws
   source devel/setup.sh
   roslaunch hisi_demo hisi_demo.launch
   ```
10. 在主机上订阅消息
    ```
    export ROS_MASTER_URI=http://hisi:11311/
    rostopic echo /chatter
    ```