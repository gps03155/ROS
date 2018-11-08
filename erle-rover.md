# Erle-Rover 환경설정
---

<br><br>

>### __Erele-Brain ip 설정 및 ssh 연결__

~/.bashrc 파일에서 ROS_MASTER_URI와 ROS_HOSTNAME IP를 수정해야함
roscore는 로봇이 켜짐과 동시에 실행되므로 따로 작동시키지 않아도 됨
roscore가 계속 켜져있기 때문에 .launch 파일을 수정할 경우 로봇을 껐다 켜야 적용됨

<pre>
ROS_MASTER_URI = erle-brain3의 ip
ROS_HOSTNAME = 내 pc ip

설정 후 ssh 연결

$ ssh erle@ip // 초기 비밀번호 : holaerle

$ rosnode list
$ rostopic list
</pre>
docs와 나타나는 node, topic이 다를 수 있음

<br><br><br>

>### __mavros 설치__

ROS를 기반으로 로봇을 제어하기 위해서는 mavros가 필수적임

<pre>
$ cd /opt/ros/kinetic/setup_mavros.bash

bash 파일의 fcu_url과 gcs_url은 ip 설정해줘야함

$ rosrun mavros mavros_node

GCS 변수 값 확인
$ rosrun mavros mavparam get ARMING_CHECK   // 0
                             ARMING_REQUIRE // 0
                             SYSID_MYGCS    // 1

GCS 변수 값 변경할 경우
$ rosrun mavros mavparam set 변수명 값

</pre>
로봇을 움직이고자 할 때는 항상 mavros가 실행 상태여야함

<br>

- 로봇이 움직이지 않을 경우
/mvaros/rc/in 토픽의 채널 값 확인
<pre>
$ rostopic echo /mavros/rc/in

channels = [] 이라고 잡히면 컨트롤러 ON

재구동 후 확인 시 channels의 값이 잡힌 것을 확인할수 있음
</pre>

<br><br><br>

>## __카메라 사용__

- 이미지 캡쳐

<pre>
$ sudo raspistill -o erlepic.png
</pre>

<br>

- 카메라 영상을 Rviz에서 확인
Rviz에서 add → image 선택하면 영상 확인가능
/camera/image 토픽 이용

<pre>
$ rosservice call /camera/start_capture // 카메라 켤 때
$ rosservice call /camera/stop_capture // 카메라 끌 때

$ git clone https://github.com/ros-perception/image_pipeline.git
$ rosrun image_view image_view image:=/camera/image _image_transport:=compressed
</pre>


<br>

- topic 값이 넘어오지 않을 경우
/etc/hosts 파일 마지막에 아래 행 추가
<pre>
echo -e "erle-brain3의 IP\terle-brain"
</pre>

<br><br><br>

>### __Teleoperating__

workspace를 생성 후 예제 코드 실행 ( 키보드 조작 )
기존 소스코드에는 A, B, C, D로 되어있음 ( 방향키도 가능 )

<pre>
$ mkdir -p ~/erle_ws/src
$ cd ~/erle_ws/src
$ cd ..
$ catkin_make --pkg ros_erle_cpp_teleoperation_erle_rover // 에러 시 catkin_make만 실행

$ rosrun ros_erle_cpp_teleoperation_erle_rover teleoperation
$ rostopic echo /mavros/rc/in
$ rostopic echo /mavros/rc/orverride // 속도, 방향 변화값 확인 가능
</pre>

<br>

- catkin_make 에러 날 경우
<pre>
1. libmavconn, mavros_msg
$ git clone https://github.com/erlerobot/mavros.git

2. eigen_conversions
$ git clone https://github.com/ros/geometry.git

// FindEigen3.cmake error
find_package(PkgConfig)
pkg_search_module(Eigen3 REQUIRED eigen3)
</pre>

<br>

- 속도 변경할 경우
Teleoperating 소스에서 main.cpp를 수정

<pre>
linear_vel_max, linear_vel_min   = 속도
angular_vel_max, angular_vel_min = 각도

channels[0] = roll     // 각도
channels[1] = pitch
channels[2] = throttle // 속도
channels[3] = yaw

소스 수정 후 빌드를 다시 해줘야함

$ catkin_make
</pre>

다른 채널값은 기본 0으로 지정
채널 값을 일반적으로 1000 ~ 2000

<br><br><br>

>### __Lidar 센서__

Lidar 센서값 확인

<pre>
$ git clone https://github.com/Slamtec/rplidar_ros.git

lidar와 로봇 연결 후

$ ls -l /dev |grep ttyUSB
$ sudo chmod 666 /dev/ttyUSB0 // ttyUSB1일 수도 있음

1. Rviz에서 확인
$ roslaunch rplidar_ros view_rplidar.launch

2. scan 값만 확인
$ roslaunch rplidar_ros rplidar.roslaunch
$ rosrun rplidar_rplidarNodeClient
</pre>

ttyUSB 포트 에러가 발생한다면 .launch 파일에서 /dev/ttyUSB 부분 수정해야함
Lidar를 사용하는 도중 No space left on device가 나타나더라도 정상 실행됨
추후 용량을 정리해주면 해결 가능

<br>

- __Point 조정__
Lidar의 인식 범위를 늘리거나 줄이기 위해 조정
point 수가 높을수록 성능이 올라감

<pre>
.launch 파일에 scan_mode 추가하여 수정

1. Standard : 2.0k
2. Express  : 4.0k
3. Boost    : 8.0k

param name="scan_mode" type="string" value="scan_mode명"
</pre>
