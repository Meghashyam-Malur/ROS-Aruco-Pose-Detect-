## Start ROS:

`roscore`

## opening simple usbcam node:

`rosrun usb_cam usb_cam_node`

## performing camera calibration:

`rosrun camera_calibration cameracalibrator.py --size 9x6 --square 0.0200 image:=/usb_cam/image_raw camera:=/usb_cam --no-service-check`

Note: --size parameter need to be changed in accordance with the (m-1)x(n-1)values for an m x n grid used for calibration
      --square parameter needs to be changed in accordance with the side lenght of each square of grid here 200mm
      --no-service-check is to prevent warnings due to absence of calibration filewhen calibration is being performed for the first time

Note:  has been done  as of 7/10/22 need not be done again

### Result For reference:

**** Calibrating ****
mono pinhole calibration...
D = [0.08959309131208465, 0.1927316472201394, 0.002308237487304637, 0.003922913033686664, 0.0]
K = [828.9369597414833, 0.0, 318.74153675428596, 0.0, 831.3300981774705, 232.33240733025008, 0.0, 0.0, 1.0]
R = [1.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 1.0]
P = [854.5978393554688, 0.0, 320.3380834205891, 0.0, 0.0, 857.84619140625, 233.2079072136985, 0.0, 0.0, 0.0, 1.0, 0.0]
None
#oST version 5.0 parameters


[image]

width
640

height
480

[narrow_stereo]

camera matrix
828.936960 0.000000 318.741537
0.000000 831.330098 232.332407
0.000000 0.000000 1.000000

distortion
0.089593 0.192732 0.002308 0.003923 0.000000

rectification
1.000000 0.000000 0.000000
0.000000 1.000000 0.000000
0.000000 0.000000 1.000000

projection
854.597839 0.000000 320.338083 0.000000
0.000000 857.846191 233.207907 0.000000
0.000000 0.000000 1.000000 0.000000

('Wrote calibration data to', '/tmp/calibrationdata.tar.gz')
D = [0.08959309131208465, 0.1927316472201394, 0.002308237487304637, 0.003922913033686664, 0.0]
K = [828.9369597414833, 0.0, 318.74153675428596, 0.0, 831.3300981774705, 232.33240733025008, 0.0, 0.0, 1.0]
R = [1.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 1.0]
P = [854.5978393554688, 0.0, 320.3380834205891, 0.0, 0.0, 857.84619140625, 233.2079072136985, 0.0, 0.0, 0.0, 1.0, 0.0]
#oST version 5.0 parameters


[image]

width
640

height
480

[narrow_stereo]

camera matrix
828.936960 0.000000 318.741537
0.000000 831.330098 232.332407
0.000000 0.000000 1.000000

distortion
0.089593 0.192732 0.002308 0.003923 0.000000

rectification
1.000000 0.000000 0.000000
0.000000 1.000000 0.000000
0.000000 0.000000 1.000000

projection
854.597839 0.000000 320.338083 0.000000
0.000000 857.846191 233.207907 0.000000
0.000000 0.000000 1.000000 0.000000


## Setting up Ros Single Aruco Detection node:

### Usb Camera stream publisher Launch file: 

usb_cam_stream_publisher.launch

```ROS
<launch>
<arg name="video_device" default="/dev/video0" />
<arg name="image_width" default="640" />
<arg name="image_height" default="480" />

<node name="usb_cam" pkg="usb_cam" type="usb_cam_node" output="screen" >
<param name="video_device" value="$(arg video_device)" />
<param name="image_width" value="$(arg image_width)" />
<param name="image_height" value="$(arg image_height)"/>
<param name="pixel_format" value="mjpeg" />
<param name="camera_frame_id" value="usb_cam" />
<param name="io_method" value="mmap"/>
</node>
</launch>
```

### Aruco Marker Detection and Pose Estimation Launch file:

Aruco_marker_finder.launch

```
<launch>

<arg name="markerId" default="701"/>
<arg name="markerSize" default="0.05"/> <!-- in meter -->
<arg name="eye" default="left"/>
<arg name="marker_frame" default="marker_frame"/>
<arg name="ref_frame" default=""/> <!-- leave empty and the pose will be published wrt param parent_name -->
<arg name="corner_refinement" default="LINES" /> <!-- NONE, HARRIS, LINES, SUBPIX -->

<node pkg="aruco_ros" type="single" name="aruco_single">
<remap from="/camera_info" to="/usb_cam/camera_info" />
<remap from="/image" to="/usb_cam/image_raw" />
<param name="image_is_rectified" value="True"/>
<param name="marker_size" value="$(arg markerSize)"/>
<param name="marker_id" value="$(arg markerId)"/>
<param name="reference_frame" value="$(arg ref_frame)"/> <!-- frame in which the marker pose will be refered -->
<param name="camera_frame" value="base_link"/>
<param name="marker_frame" value="$(arg marker_frame)" />
<param name="corner_refinement" value="$(arg corner_refinement)" />
</node>

</launch>
```

##  Startup Procedure:

In multiple terminals opened in same location as above launch files run following commands in same order:

Terminal 1: 

`roslaunch usb_cam_stream_publisher.launch`

Terminal 2: 

`roslaunch aruco_marker_finder.launch markerId:=701 markerSize:=0.05`

Note: markerId parameter to define which specific marker to look for
      markerSize parameter to define side length of aruco marker to look for
      if the above two parameters are not passed, default values will be used as defined in the launch files

Terminal 3:

`rosrun rqt_gui rqt_gui`

Note: in rqt gui open image view which can be found under Plugins > Vizualizations > Image View  to see results

Terminal 4: 

`rostopic echo /aruco_single/pose //To view the Aruco marker position and orientation`


