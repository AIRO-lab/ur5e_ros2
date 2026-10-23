# UR5e · DH 그리퍼 노트북 실습 가이드

Ubuntu 22.04 · ROS 2 Humble · MoveIt 2 · AG-160-95-W-S

편집: 2026-10-23. 학생이 자신의 노트북에서 UR5e와 DH Robotics 그리퍼를 사용할 수 있도록 환경을 준비하는 수업 자료다. 로봇과 그리퍼의 장착·배선은 완료된 상태를 사용한다.

<!-- chapter:start|시작하기 -->
# UR5e와 그리퍼를<br>내 노트북에서 사용하기

노트북 환경을 준비하고, UR5e는 MoveIt 2로, DH 그리퍼는 ROS 2 토픽 명령으로 움직여 본다.

<div class="tags"><span>Ubuntu 22.04</span><span>ROS 2 Humble</span><span>UR5e</span><span>DH AG-160-95-W-S</span></div>

<!-- hero -->

## 실습 장비

| 구성 | 사용하는 장비·설정 |
|---|---|
| 로봇팔 | Universal Robots UR5e |
| 그리퍼 | DH Robotics AG-160-95-W-S |
| 노트북 | Ubuntu 22.04 · ROS 2 Humble 설치 완료 |
| 노트북 유선 IP | `192.168.0.1` |
| 로봇 IP | `192.168.0.2` |
| 그리퍼 통신 | 노트북의 USB–RS485 어댑터 |
| 그리퍼 장치 경로 | `/dev/robot/dh_ag95_gripper` |

노트북에는 로봇용 LAN과 그리퍼용 USB 어댑터를 연결한다. 로봇과 그리퍼의 기존 장착·배선은 그대로 사용한다.

## 실습 순서

**노트북 준비 → 통신 설정 → 로봇 켜기 → UR5e·MoveIt 실행 → 그리퍼 장치 고정 → 그리퍼 실행** 순서로 진행한다. 설치를 마친 뒤에는 마지막 [정리](#daily)에서 실행 명령을 한 번에 확인할 수 있다.

사진과 영상은 HTML 안에 포함되어 있다. 이 파일 하나를 공유하면 오프라인에서도 자료를 볼 수 있다. 영상의 미리보기에서 **영상 재생**을 누른다.

<!-- chapter:setup|노트북 준비 -->
# 노트북 준비

ROS 2 Humble이 설치된 Ubuntu 22.04 노트북을 기준으로 한다. `{your_ros2_workspace}`는 새로 만들 작업공간 이름으로 바꾼다. 예를 들어 `ur5e_ws`를 사용할 수 있다.

## 1. ROS·UR·MoveIt·개발 도구

```bash
sudo apt install \
  ros-humble-ur ros-humble-moveit \
  ros-humble-ros2-control ros-humble-ros2-controllers \
  ros-humble-xacro ros-humble-robot-state-publisher \
  python3-colcon-common-extensions python3-rosdep \
  python3-vcstool git build-essential cmake
```

## 2. 수업용 패키지 받기

수업용 통합 저장소에는 UR 드라이버, 수정된 그리퍼 패키지와 serial 의존성이 함께 들어 있다. 학생은 아래 **한 번의 clone**으로 소스를 받는다. 패키지가 중복되지 않도록 새 작업공간을 사용한다.

<div class="note">수업용 저장소: <a href="https://github.com/AIRO-lab/ur5e_ros2">AIRO-lab/ur5e_ros2</a>. 아래 명령으로 작업공간의 <code>src/ur5e_ros2</code> 폴더에 내려받는다.</div>

```bash
mkdir -p ~/{your_ros2_workspace}/src
cd ~/{your_ros2_workspace}/src
git clone https://github.com/AIRO-lab/ur5e_ros2.git
```

rosdep을 처음 사용하는 노트북에서는 한 번 초기화한다. 이미 초기화했다면 `rosdep update`만 실행한다.

```bash
sudo rosdep init
rosdep update
```

필요한 의존성을 설치하고 작업공간을 빌드한다.

```bash
source /opt/ros/humble/setup.bash
cd ~/{your_ros2_workspace}
rosdep install --from-paths src --ignore-src --rosdistro humble -y
colcon build --symlink-install
```

## 3. 터미널 환경

빌드를 마친 뒤, **새로 여는 각 터미널에서** 다음 명령을 실행한다.

```bash
source ~/{your_ros2_workspace}/install/setup.bash
```

<!-- chapter:network|노트북 통신 설정 -->
# 노트북 통신 설정

노트북의 유선 IP를 설정하고, 로봇과 그리퍼 USB가 연결되어 있는지 확인한다.

## 유선 IPv4 설정

Ubuntu **설정 → 네트워크 → 유선 연결의 설정 → IPv4 → 수동**에서 입력한다.

| 항목 | 설정값 |
|---|---|
| 노트북 주소 | `192.168.0.1` |
| 넷마스크 | `255.255.255.0` |
| 로봇 주소 | `192.168.0.2` |
| 게이트웨이·DNS | 로봇 전용 직결망에서는 비워 둠 |

<!-- figure:ip-manual -->

캡처의 게이트웨이 칸은 예시와 달리 **비워 두고** 적용한다. 노트북 IP `.1`과 로봇 IP `.2`는 서로 다르게 설정한다.

## 로봇 연결 확인

```bash
ping 192.168.0.2
```

사진처럼 `bytes from 192.168.0.2`가 반복해서 나오면 로봇과 통신하고 있는 것이다. `Ctrl+C`로 종료한다.

<!-- figure:ping -->

## 그리퍼 USB 인식

```bash
lsusb
ls -l /dev/serial/by-id/
```

아래 사진의 **FT232 Serial (UART) IC**와 **BG02Q5ZU**가 포함된 장치가 이번 그리퍼의 USB 어댑터다.

<!-- figure:usb-device -->

사진에서는 `/dev/ttyUSB0`로 연결되어 있다. USB 번호는 바뀔 수 있으므로 [그리퍼 장치 고정](#gripper)에서 실행에 사용할 이름을 지정한다.

<!-- chapter:power|로봇 켜기와 모드 -->
# 로봇 켜기와 모드

로봇과 그리퍼 주변을 비운 뒤, 기존에 설정된 로봇 설치·도구 설정을 사용한다.

## 로봇 켜기

1. 제어기 전원을 켜고 펜던트의 로봇 초기화 화면을 연다.
2. 비상정지 원인이 해소된 것을 확인하고 해제한 뒤 **ON**을 누른다.
3. **START**를 눌러 브레이크를 해제한다.

<!-- figure:robot-init -->

사진처럼 **Brakes Released**, **Robot in Normal Mode**와 초록색 상태가 표시되면 로봇 켜기가 끝난 것이다. 이제 노트북에서 드라이버를 실행할 수 있다.

## Remote Control과 Headless

이번 로봇은 **Remote Control 기능이 이미 활성화되어 있다.** 로봇을 켠 뒤 운용 모드가 **Remote Control**인지 확인한다. Headless는 노트북의 드라이버가 로봇 제어 프로그램을 보내 실행하는 방식이다.

## 그리퍼 LED

| LED 표시 | 의미 |
|---|---|
| 빨강 깜빡임 | 초기화 전 — 그리퍼 드라이버를 실행하기 전의 상태 |
| 파랑 지속 점등 | 초기화 완료 |
| 빨강 짧은 점멸 | 명령 수신 |
| 초록 지속 점등 | 물체 파지 |
| 초록 깜빡임 | 물체 이탈 |

그리퍼 초기화는 뒤의 **그리퍼 실행 명령**에서 진행된다.

<!-- chapter:arm|UR5e · MoveIt 2 -->
# UR5e와 MoveIt 2 실행

터미널 두 개를 열고, 각각 [터미널 환경](#setup)의 `source` 명령을 실행한 뒤 진행한다.

## 터미널 A — UR 드라이버

```bash
ros2 launch ur_robot_driver ur_control.launch.py ur_type:=ur5e robot_ip:=192.168.0.2
```

우리 로봇은 **UR5e**, IP는 **192.168.0.2**이므로 위와 같이 지정한다. 이 터미널은 실행 상태로 둔다.

수업용 통합 패키지는 **Headless가 기본으로 켜져 있어** 위 명령 그대로 사용한다. 로봇의 운용 모드는 **Remote Control**로 둔다.

## 터미널 B — MoveIt 실행

```bash
ros2 launch ur_moveit_config ur_moveit.launch.py ur_type:=ur5e launch_rviz:=true
```

이 명령으로 열린 **MotionPlanning 패널이 있는 RViz**에서 로봇을 조작한다.

## RViz에서 움직이기

<!-- figure:ur-rviz-start -->

1. Planning Group을 **`ur_manipulator`**로 선택한다. 처음에는 속도·가속도 배율을 **0.1**로 둔다.
2. 로봇 끝의 화살표를 조금 움직여 가까운 목표 위치를 지정한다.
3. **Plan**으로 경로를 확인한 뒤, 주변이 비어 있으면 **Execute**를 누른다.

<!-- figure:ur-rviz-target -->

## MoveIt 실행 화면

영상의 RViz 조작과 실행 로그를 참고한다. 실제 로봇에 연결된 상태에서 Execute하면 UR5e도 계획한 경로를 따라 움직인다.

<!-- video:ur-moveit -->

<!-- chapter:gripper|그리퍼 장치 고정 -->
# 그리퍼 장치 고정

수업용 통합 패키지의 설치·빌드는 [노트북 준비](#setup)에서 한 번만 진행한다. 여기서는 USB를 다시 꽂아도 같은 장치 이름으로 실행할 수 있도록 설정한다.

## 어댑터 serial 확인

```bash
udevadm info --name=/dev/ttyUSB0 --attribute-walk | grep serial
```

`/dev/ttyUSB0`는 앞에서 확인한 장치 경로다. 다른 번호로 연결되었다면 해당 번호로 바꾼다.

<!-- figure:usb-serial -->

이번 어댑터의 serial은 **`BG02Q5ZU`**다. 사진의 `0000:05:00.3`은 상위 장치 항목이며, 아래 규칙에는 `BG02Q5ZU`를 사용한다.

## 고정 이름 추가

```bash
sudo nano /etc/udev/rules.d/99-dh-ag95-gripper.rules
```

파일에 다음 한 줄을 넣고 저장한다.

```text
SUBSYSTEM=="tty", ATTRS{serial}=="BG02Q5ZU", ENV{ID_MM_DEVICE_IGNORE}="1", ATTR{device/latency_timer}="1", SYMLINK+="robot/dh_ag95_gripper"
```

이 규칙은 해당 어댑터에 **`/dev/robot/dh_ag95_gripper`**라는 고정 이름을 붙인다. 다른 어댑터를 사용한다면 직접 확인한 serial로 바꾼다.

그리퍼 드라이버가 종료된 상태에서 규칙을 다시 읽고, USB를 뺐다가 다시 꽂는다.

```bash
sudo udevadm control --reload-rules
```

## USB 접근 권한

새 노트북에서는 사용자를 `dialout` 그룹에 추가한 뒤 **로그아웃·로그인**한다. 이미 해당 그룹에 속해 있다면 다시 설정하지 않아도 된다.

```bash
sudo usermod -aG dialout "$USER"
```

<!-- chapter:namespaces|팔·그리퍼 제어 경로 -->
# 팔과 그리퍼의 제어 경로

UR5e와 그리퍼를 함께 실행할 수 있도록, 수정된 패키지에서는 그리퍼의 이름 앞에 **`/gripper`**를 붙여 구분했다.

| 대상 | Controller manager | 상태 토픽 |
|---|---|---|
| UR5e | `/controller_manager` | `/joint_states` |
| DH 그리퍼 | `/gripper/controller_manager` | `/gripper/joint_states` |

그리퍼 명령은 **`/gripper/forward_position_controller/commands`**로 보낸다. 수업용 패키지에 이 구성이 포함되어 있으므로 학생이 별도의 충돌 해결 패치를 적용할 필요가 없다.

현재 수정된 launch는 `gripper`를 기본 namespace로 사용한다. 다음 장의 명령에서는 `namespace:=gripper`를 따로 입력하지 않는다.

<!-- chapter:operate|그리퍼 초기화·개폐 -->
# 그리퍼 초기화와 개폐

그리퍼 손가락 사이와 주변을 비운 뒤 진행한다. 드라이버를 실행하면 초기화 과정에서 손가락이 움직일 수 있다.

## 터미널 C — 그리퍼 실행

새 터미널에서 작업공간을 `source`한 뒤 실행한다.

```bash
ros2 launch dh_gripper_driver dh_ag95_control.launch.py com_port:=/dev/robot/dh_ag95_gripper
```

드라이버가 그리퍼를 초기화하고, 아래 사진처럼 별도의 RViz 창을 연다. 초기화가 끝나면 개폐 명령을 보낼 수 있다.

<!-- figure:gripper-rviz-first -->

## 다른 터미널 — 열기·닫기

새 터미널에서도 작업공간을 `source`한다. 아래 명령을 하나씩 보내면 영상처럼 RViz 모델이 바뀌고 **연결된 실제 그리퍼도 움직인다.**

열기:

```bash
ros2 topic pub --once /gripper/forward_position_controller/commands std_msgs/msg/Float64MultiArray "{data: [0.0]}"
```

닫기:

```bash
ros2 topic pub --once /gripper/forward_position_controller/commands std_msgs/msg/Float64MultiArray "{data: [0.93]}"
```

`--once`는 명령을 한 번 발행한다. `0.0`은 열기, `0.93`은 닫기 목표값이다.

<!-- video:gripper -->

<!-- chapter:daily|정리 -->
# 정리

설치를 마친 뒤에는 로봇 전원·Remote 모드·LAN·그리퍼 USB를 확인하고, 아래 순서로 실행한다.

## 각 터미널에서 환경 불러오기

```bash
source ~/{your_ros2_workspace}/install/setup.bash
```

## 터미널 A — UR5e

```bash
ros2 launch ur_robot_driver ur_control.launch.py ur_type:=ur5e robot_ip:=192.168.0.2
```

수업용 통합 패키지의 기본값이 `headless_mode:=true`이므로 추가 인자를 입력하지 않아도 된다.

## 터미널 B — MoveIt 2

```bash
ros2 launch ur_moveit_config ur_moveit.launch.py ur_type:=ur5e launch_rviz:=true
```

## 터미널 C — 그리퍼

```bash
ros2 launch dh_gripper_driver dh_ag95_control.launch.py com_port:=/dev/robot/dh_ag95_gripper
```

팔은 MoveIt의 **Plan → Execute**, 그리퍼는 [열기·닫기 토픽 명령](#operate)으로 동작시킨다. 실습을 마치면 이동을 마친 상태에서 각 launch 터미널을 `Ctrl+C`로 종료한다.

## 다음 단계 — Pick & Place

현재 구조는 **UR5e 모션과 그리퍼를 각각 ROS 2 launch로 실행**한다. 이후 MoveIt Setup Assistant로 팔과 그리퍼의 모델·설정을 통합하거나, 현재 구성을 유지하면서 두 장치의 명령을 순서대로 호출하는 코드를 작성할 수 있다.

<div class="flow"><span>집을 위치로 이동</span><b>→</b><span>그리퍼 닫기</span><b>→</b><span>놓을 위치로 이동</span><b>→</b><span>그리퍼 열기</span></div>

각 동작이 완료된 것을 확인한 뒤 다음 동작을 실행하도록 연결하면 **Pick & Place**를 구현할 수 있다. 이번 실습에서는 그 응용을 위한 노트북 환경과 팔·그리퍼의 기본 실행을 준비한다.
