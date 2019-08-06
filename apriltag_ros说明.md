# apriltag_ros说明

### 基本说明

1. ros官方文档  http://wiki.ros.org/apriltag_ros

2. github 地址 https://github.com/AprilRobotics/apriltag_ros

3. 技术说明：

   基于AprilTag库的二维tag位姿估计

   使用方便，有多个tag标签系列，可以同时识别多个标签

### 使用说明

1.  **安装：**

   + 依赖安装AprilTag 3    https://github.com/AprilRobotics/apriltag

     ```c
     $ cmake .
     $ sudo make install
     ```

   + 将apriltag_ros包下载至工作空间中src下进行编译即可

2. **使用：**

   + 调用  continuous_detection.launch

     launch中需要更改参数：相机的名称，相机的坐标系，识别的图像的话题

     ```
     <arg name="camera_name" default="/usb_cam" />
     <arg name="camera_frame" default="camera" />
     <arg name="image_topic" default="image_raw" />
     ```

     以及在tags.yaml文件中添加tag的信息（以下为标准的tag）

     ```
     standalone_tags:
       [
         {id: 1, size: 0.057, name: test1},
         {id: 0, size: 0.054, name: test0},
       ]
     ```

     在setting.yaml文件中可以设置参数：标签的family或者最多识别的标签数

   + 相机参数说明：

     调用的相机参数为camera_info话题中的相机参数信息

     ​	默认相机调用的相机参数在.ros/camera_info下

     ​    使用realsense时，相机会有自带的相机参数消息发布

     可以在launch文件中修改camera_info来源

     ```
     <remap from="camera_info" to="$(arg camera_name)/camera_info" />
     ```

   + 标签的生成：

     官方链接为<https://april.eecs.umich.edu/software/apriltag.html>[  但是没有找到](USER_CANCEL)

     使用OpenMV生成 <http://book.openmv.cc/image/apriltag.html>

   + 自己添加的launch文件：

     cam_detection.launch             调用realsense的彩色相机（需要另外启动相机）

     usb_cam_detection.launch     直接调用笔记本的相机

3. **输出**

   + 图像输出：

     话题： /tag_detections_image

     在图像中显示位姿检测结果

     可以在launch文件后添加图像显示节点启动

     ```html
     <!--start image View visual detection result-->
     <node name="image_view" pkg="image_view" type="image_view" respawn="false" output="screen">
         <remap from="image" to="/tag_detections_image"/>
         <param name="autosize" value="true" />
     </node>
     ```

   + 位姿检测结果输出：

     话题：/tag_detections
     
     图中0和1Tag在tags.yaml中进行说明，因此能检测到
     
     ![1560584303501](/home/sujie/.config/Typora/typora-user-images/1560584303501.png)
   
   + 话题发布的消息包含的内容在msg文件夹下
   
     AprilTagDetection.msg中查看
   
     对比于源代码，添加了输出中心点及四个角点的坐标
   
     ```
     int16[]  center_point
     int16[]  left_top
     int16[]  right_top
     int16[]  right_down
     int16[]  left_down
     ```
   
     相应的在common_function.cpp中297行添加了代码：
   
     ```
     //    Add one center and four point coordinates
         tag_detection.center_point.push_back((int)detection->c[0]);
         tag_detection.center_point.push_back((int)detection->c[1]);
         tag_detection.left_top.push_back((int)detection->p[0][0]);
         tag_detection.left_top.push_back((int)detection->p[0][1]);
         tag_detection.right_top.push_back((int)detection->p[3][0]);
         tag_detection.right_top.push_back((int)detection->p[3][1]);
         tag_detection.right_down.push_back((int)detection->p[2][0]);
         tag_detection.right_down.push_back((int)detection->p[2][1]);
         tag_detection.left_down.push_back((int)detection->p[1][0]);
         tag_detection.left_down.push_back((int)detection->p[1][1]);
     ```
   
     



<p align="right">2019.06.15苏杰</p>