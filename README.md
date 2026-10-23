# UR5e + DH 그리퍼 실습 패키지

Ubuntu 22.04 / ROS 2 Humble에서 UR5e와 DH Robotics AG-160-95-W-S를 사용하는 수업용 통합 소스다. UR 드라이버, 그리퍼 드라이버와 serial 라이브러리를 한 저장소에 포함했다.

수업용 저장소: [AIRO-lab/ur5e_ros2](https://github.com/AIRO-lab/ur5e_ros2)

## 포함된 구성

| 폴더 | 내용 |
|---|---|
| `Universal_Robots_ROS2_Driver/` | UR 드라이버·MoveIt 설정, `headless_mode` 기본값 `true` |
| `dh_ag95_gripper_ros2/` | `/gripper` namespace 분리를 적용한 그리퍼 드라이버 |
| `serial-ros2/` | 그리퍼 USB 통신 라이브러리 |
| `docs/` | 학생용 HTML·Markdown 실습 가이드 |

세 소스 폴더는 일반 파일로 포함되어 있다. 서브모듈 설정이나 별도의 추가 clone은 필요 없다. 이 통합본의 실행·설치 절차는 **이 README와 `docs/` 가이드**를 따른다. 각 소스 폴더의 기존 README는 원본 참고 자료로 보존했다.

## 설치

ROS 2 Humble이 설치된 Ubuntu 22.04에서 진행한다. 기존 `ros_ur_driver/src`에는 같은 패키지가 있으므로, 아래처럼 **새 작업공간**에서 빌드한다.

```bash
sudo apt install \
  ros-humble-ur ros-humble-moveit \
  ros-humble-ros2-control ros-humble-ros2-controllers \
  ros-humble-xacro ros-humble-robot-state-publisher \
  python3-colcon-common-extensions python3-rosdep \
  python3-vcstool git build-essential cmake

mkdir -p ~/ur5e_ws/src
cd ~/ur5e_ws/src
git clone https://github.com/AIRO-lab/ur5e_ros2.git
```

rosdep을 처음 사용하는 노트북은 `sudo rosdep init`을 한 번 실행한다. 이미 초기화했다면 생략한다.

```bash
source /opt/ros/humble/setup.bash
rosdep update
cd ~/ur5e_ws
rosdep install --from-paths src --ignore-src --rosdistro humble -y
colcon build --symlink-install
```

## 실행

로봇 IP는 `192.168.0.2`, 노트북 유선 IP는 `192.168.0.1`을 사용한다. 로봇을 켜고 **Remote Control** 모드로 둔다. 그리퍼 장치 고정과 USB 권한 설정은 [실습 가이드](docs/UR5e_DH_노트북_실습가이드.md#그리퍼-장치-고정)를 따른다.

**각 터미널에서 먼저** 새로 빌드한 작업공간을 불러온다.

```bash
source ~/ur5e_ws/install/setup.bash
```

터미널 A — UR5e:

```bash
ros2 launch ur_robot_driver ur_control.launch.py ur_type:=ur5e robot_ip:=192.168.0.2
```

이 통합본은 `headless_mode` 기본값이 `true`이므로 위 명령으로 Headless 연결을 사용한다. 기존 원본 작업공간이나 apt 설치본의 기본값을 변경한 것은 아니다. 반드시 **통합본을 빌드한 작업공간**을 source한다.

터미널 B — MoveIt 2:

```bash
ros2 launch ur_moveit_config ur_moveit.launch.py ur_type:=ur5e launch_rviz:=true
```

터미널 C — 그리퍼:

```bash
ros2 launch dh_gripper_driver dh_ag95_control.launch.py com_port:=/dev/robot/dh_ag95_gripper
```

기본 namespace는 `gripper`다. `namespace:=gripper`를 추가하지 않아도 팔과 그리퍼의 controller manager가 분리된다. 그리퍼 launch는 초기화 중 손가락을 움직일 수 있으므로 동작 공간을 비운 뒤 실행한다.

다른 터미널에서 열기·닫기:

```bash
# 열기
ros2 topic pub --once /gripper/forward_position_controller/commands std_msgs/msg/Float64MultiArray "{data: [0.0]}"

# 닫기
ros2 topic pub --once /gripper/forward_position_controller/commands std_msgs/msg/Float64MultiArray "{data: [0.93]}"
```

팔과 그리퍼는 각각 제어한다. 이 저장소는 두 장치가 합쳐진 MoveIt 모델이나 자동 Pick & Place 코드를 제공하지 않는다. 후속 실습에서 모델·설정을 통합하거나 동작 완료를 확인하며 명령을 순서대로 호출하는 코드를 작성한다.

## 실습 자료

- [HTML 실습 가이드](docs/UR5e_DH_노트북_실습가이드.html): 파일을 내려받아 브라우저에서 연다. 사진·영상이 HTML에 포함되어 있다.
- [Markdown 실습 가이드](docs/UR5e_DH_노트북_실습가이드.md): 내용과 명령 편집용 원본. 미디어는 HTML 파일에서 확인한다.
