# RealSense D435 x2 ROS2 Setup Guide

이 문서는 Intel RealSense D435 카메라 2대를 ROS2 Humble 환경에서 RGB-only 모드로 안정적으로 사용하기 위한 설치 및 실행 가이드입니다.

## 1. 목표 구성

본 설정의 목표는 다음과 같습니다.

- RealSense D435 카메라 2대 사용
  - VR / Top 카메라
  - Robot / End-effector 카메라
- RGB 영상만 사용
  - Depth OFF
  - Infrared OFF
  - Gyro / Accel OFF
  - PointCloud OFF
- USB 대역폭 문제를 줄이기 위해 Robot 카메라는 낮은 FPS 또는 낮은 해상도 사용
- ROS2 topic을 통해 카메라 영상을 확인

예상 장치 구성 예시는 다음과 같습니다.

```bash
rs-enumerate-devices -s
```

예시 출력:

```text
Device Name              Serial Number   Firmware Version
Intel RealSense D435     244222070489    5.17.0.9
Intel RealSense D435     243622073271    5.15.1.55
```

현재 스크립트 기준 매칭은 다음과 같습니다.

| Camera role | Serial number |
|---|---:|
| VR / Top camera | `243622073271` |
| Robot / End-effector camera | `244222070489` |

> 실제 물리적으로 어떤 카메라가 VR이고 Robot인지 반드시 한 번 확인해야 합니다.

---

## 2. 필요한 패키지

ROS2 Humble 기준으로 다음 패키지가 필요합니다.

### 2.1 RealSense SDK / librealsense

RealSense 카메라를 리눅스에서 인식하고 `rs-enumerate-devices`, `realsense-viewer` 등을 사용하기 위해 필요합니다.

```bash
sudo apt update
sudo apt install -y ros-humble-librealsense2*
```

설치 확인:

```bash
rs-enumerate-devices
realsense-viewer
```

---

### 2.2 ROS2 RealSense wrapper

다음 명령을 사용하려면 `realsense2_camera` ROS2 패키지가 필요합니다.

```bash
ros2 launch realsense2_camera rs_launch.py
```

설치:

```bash
sudo apt install -y ros-humble-realsense2-*
```

설치 확인:

```bash
ros2 pkg list | grep realsense
```

정상적으로 설치되었다면 다음과 같은 패키지가 보여야 합니다.

```text
realsense2_camera
realsense2_camera_msgs
```

---

### 2.3 image_tools

스크립트에서 다음 명령을 사용합니다.

```bash
ros2 run image_tools showimage
```

이를 위해 `image_tools` 패키지가 필요합니다.

```bash
sudo apt install -y ros-humble-image-tools
```

실행 확인:

```bash
ros2 run image_tools showimage
```

---

### 2.4 rqt_image_view

`rqt_image_view`로 영상을 확인하려면 다음 패키지를 설치합니다.

```bash
sudo apt install -y ros-humble-rqt-image-view
```

실행:

```bash
rqt_image_view
```

단, RealSense image topic이 회색 화면으로 보이거나 QoS 문제로 표시되지 않는 경우가 있으므로, 단순 확인용으로는 `showimage` 사용을 권장합니다.

---

## 3. 전체 설치 명령어

한 번에 설치하려면 다음 명령을 사용합니다.

```bash
sudo apt update
sudo apt install -y \
  ros-humble-librealsense2* \
  ros-humble-realsense2-* \
  ros-humble-image-tools \
  ros-humble-rqt-image-view
```

ROS2 환경을 source 합니다.

```bash
source /opt/ros/humble/setup.bash
```

매번 터미널을 열 때 자동으로 적용하려면 `~/.bashrc`에 추가할 수 있습니다.

```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

---

## 4. 설치 후 기본 확인 순서

### 4.1 카메라 장치 인식 확인

```bash
rs-enumerate-devices
```

또는 serial number만 간단히 확인:

```bash
rs-enumerate-devices -s
```

카메라 2대가 모두 표시되어야 합니다.

---

### 4.2 USB 연결 상태 확인

```bash
lsusb -t
```

Robot / End-effector 카메라가 USB2.x로 연결되어 있다면 고해상도/고FPS에서 불안정할 수 있습니다.

권장 설정:

| Camera | Recommended profile |
|---|---|
| VR / Top, USB3 | `640x480x30` |
| Robot / EE, USB2.x | `640x480x15` |
| Robot / EE fallback | `424x240x30` |

---

### 4.3 RealSense ROS2 기본 실행 확인

```bash
ros2 launch realsense2_camera rs_launch.py
```

다른 터미널에서 topic 확인:

```bash
ros2 topic list | grep camera
```

---

## 5. 권장 실행 방식

카메라 2대를 사용할 때는 각각 다른 터미널에서 실행하는 것을 권장합니다.

### Terminal 1: VR / Top camera

```bash
rs_vr
```

### Terminal 2: Robot / End-effector camera

```bash
rs_robot
```

이 방식은 각 카메라의 로그와 에러를 따로 확인할 수 있어 디버깅에 유리합니다.

---

## 6. 카메라 영상 확인

### 6.1 Topic 목록 확인

```bash
rs_topics
```

예상 topic 예시:

```text
/realsense/vr/color/image_raw
/realsense/robot/color/image_raw
```

---

### 6.2 Publish 주파수 확인

VR 카메라:

```bash
rs_hz_vr
```

Robot 카메라:

```bash
rs_hz_robot
```

직접 명령어로 확인하려면:

```bash
ros2 topic hz /realsense/vr/color/image_raw --window 5
ros2 topic hz /realsense/robot/color/image_raw --window 5
```

---

### 6.3 영상 확인

VR 카메라:

```bash
rs_view_vr
```

Robot 카메라:

```bash
rs_view_robot
```

직접 명령어로 확인하려면:

```bash
ros2 run image_tools showimage --ros-args -r image:=/realsense/vr/color/image_raw
ros2 run image_tools showimage --ros-args -r image:=/realsense/robot/color/image_raw
```

---

## 7. 현재 스크립트에서 주의할 점

기존에 작성된 bash 스크립트는 의도는 맞지만, 그대로 실행하면 에러가 날 수 있습니다.

특히 다음 문제를 수정해야 합니다.

### 7.1 export 문법 오류

잘못된 예:

```bash
export  RS_VR_PROFILE_DEFAULT = " 640x480x30"
```

올바른 예:

```bash
export RS_VR_PROFILE_DEFAULT="640x480x30"
```

bash에서는 `=` 앞뒤에 공백이 있으면 안 됩니다.

---

### 7.2 local 변수 선언 오류

잘못된 예:

```bash
local  profile = " ${1 :- $RS_VR_PROFILE_ DEFAULT } "
```

올바른 예:

```bash
local profile="${1:-$RS_VR_PROFILE_DEFAULT}"
```

---

### 7.3 변수명 중간 공백 오류

잘못된 예:

```bash
$RS_VR_PROFILE_ DEFAULT
$RS_ROBOT_ PROFILE_DEFAULT
```

올바른 예:

```bash
$RS_VR_PROFILE_DEFAULT
$RS_ROBOT_PROFILE_DEFAULT
```

---

### 7.4 topic 경로 공백 오류

잘못된 예:

```bash
"/${ RS_NS }/robot/color/image_ raw"
```

올바른 예:

```bash
"/${RS_NS}/robot/color/image_raw"
```

---

### 7.5 Robot 카메라 기본 FPS 설정 불일치

설명에서는 Robot 카메라가 USB2.x 환경이라 `640x480x15`가 안정적이라고 되어 있지만, 기존 코드에는 `640x480x30`으로 되어 있었습니다.

Robot 카메라가 USB2.x라면 아래처럼 설정하는 것을 권장합니다.

```bash
export RS_ROBOT_PROFILE_DEFAULT="640x480x15"
```

더 안정적인 fallback은 다음입니다.

```bash
rs_robot 424x240x30
```

---

## 8. 수정된 bash 설정 예시

아래 코드는 `~/.bashrc`에 추가하거나 별도 파일로 저장한 뒤 source 해서 사용할 수 있습니다.

예시:

```bash
nano ~/.realsense_aliases.sh
```

내용 추가 후:

```bash
source ~/.realsense_aliases.sh
```

자동 적용하려면:

```bash
echo "source ~/.realsense_aliases.sh" >> ~/.bashrc
source ~/.bashrc
```

수정된 예시 코드는 다음과 같습니다.

```bash
# ==============================
# RealSense D435 x2 RGB-only setup
# ==============================

export RS_VR_PROFILE_DEFAULT="640x480x30"
export RS_ROBOT_PROFILE_DEFAULT="640x480x15"
export RS_NS="realsense"

_rs_common_args() {
  cat << 'ARGS_EOF'
enable_color:=true
enable_depth:=false
enable_infra1:=false
enable_infra2:=false
enable_gyro:=false
enable_accel:=false
pointcloud.enable:=false
align_depth.enable:=false
colorizer.enable:=false
ARGS_EOF
}

rs_vr() {
  local profile="${1:-$RS_VR_PROFILE_DEFAULT}"
  ros2 launch realsense2_camera rs_launch.py \
    camera_namespace:="$RS_NS" \
    camera_name:=vr \
    serial_no:="'243622073271'" \
    $(_rs_common_args) \
    rgb_camera.profile:="$profile"
}

rs_robot() {
  local profile="${1:-$RS_ROBOT_PROFILE_DEFAULT}"
  ros2 launch realsense2_camera rs_launch.py \
    camera_namespace:="$RS_NS" \
    camera_name:=robot \
    serial_no:="'244222070489'" \
    $(_rs_common_args) \
    rgb_camera.profile:="$profile"
}

rs_robot_fallback() {
  rs_robot "424x240x30"
}

rs_both_bg() {
  echo "[rs_both_bg] launching VR(${RS_VR_PROFILE_DEFAULT}) + Robot(${RS_ROBOT_PROFILE_DEFAULT}) in background..."
  (rs_vr "$RS_VR_PROFILE_DEFAULT") & export RS_VR_PID=$!
  sleep 1
  (rs_robot "$RS_ROBOT_PROFILE_DEFAULT") & export RS_ROBOT_PID=$!
  echo "VR PID=${RS_VR_PID}, Robot PID=${RS_ROBOT_PID}"
  echo "Stop: rs_stop_bg"
}

rs_stop_bg() {
  echo "[rs_stop_bg] stopping..."
  [ -n "${RS_VR_PID:-}" ] && kill "${RS_VR_PID}" 2>/dev/null
  [ -n "${RS_ROBOT_PID:-}" ] && kill "${RS_ROBOT_PID}" 2>/dev/null
}

rs_topics() {
  ros2 topic list | grep "^/${RS_NS}/"
}

rs_hz_vr() {
  ros2 topic hz "/${RS_NS}/vr/color/image_raw" --window 5
}

rs_hz_robot() {
  ros2 topic hz "/${RS_NS}/robot/color/image_raw" --window 5
}

rs_view_vr() {
  ros2 run image_tools showimage --ros-args -r image:=/${RS_NS}/vr/color/image_raw
}

rs_view_robot() {
  ros2 run image_tools showimage --ros-args -r image:=/${RS_NS}/robot/color/image_raw
}

rs_enum() {
  rs-enumerate-devices | sed -n '1,160p'
}

alias rsv='rs_vr'
alias rsr='rs_robot'
alias rsls='rs_topics'
alias cam='python3 ~/nrs_lab2/nrs_imitation/custom/check_cam_serial.py'
```

---

## 9. 권장 사용 흐름

### 9.1 최초 확인

```bash
rs_enum
```

또는:

```bash
rs-enumerate-devices -s
```

카메라 2대의 serial number가 정상적으로 보이는지 확인합니다.

---

### 9.2 VR 카메라 실행

Terminal 1:

```bash
rs_vr
```

또는 alias 사용:

```bash
rsv
```

---

### 9.3 Robot 카메라 실행

Terminal 2:

```bash
rs_robot
```

또는 alias 사용:

```bash
rsr
```

---

### 9.4 Robot 카메라가 불안정할 때

Robot 카메라가 끊기거나 gray screen, timeout, stuttering이 발생하면 다음을 사용합니다.

```bash
rs_robot_fallback
```

이는 Robot 카메라를 다음 profile로 낮춥니다.

```text
424x240x30
```

---

### 9.5 topic 확인

```bash
rs_topics
```

---

### 9.6 Hz 확인

```bash
rs_hz_vr
rs_hz_robot
```

---

### 9.7 영상 확인

```bash
rs_view_vr
rs_view_robot
```

---

## 10. Troubleshooting

### 문제 1. `rs-enumerate-devices`에서 카메라가 안 보임

확인할 것:

```bash
lsusb
lsusb -t
```

가능한 원인:

- USB 케이블 문제
- USB 포트 문제
- 전원 부족
- udev rule 또는 RealSense driver 문제
- 카메라 firmware 문제

---

### 문제 2. Robot 카메라가 자주 끊김

가능한 원인:

- Robot 카메라가 USB2.x로 연결됨
- RGB 640x480@30이 USB2.x에서 불안정함
- 카메라 2대가 같은 USB hub 대역폭을 공유함

해결:

```bash
rs_robot 640x480x15
```

또는:

```bash
rs_robot_fallback
```

가능하면 Robot 카메라도 USB3 포트에 직접 연결하는 것이 좋습니다.

---

### 문제 3. `rqt_image_view`에서 회색 화면만 보임

`rqt_image_view`의 QoS 또는 rendering 문제일 수 있습니다.

대신 다음 명령으로 확인합니다.

```bash
rs_view_vr
rs_view_robot
```

또는 직접:

```bash
ros2 run image_tools showimage --ros-args -r image:=/realsense/vr/color/image_raw
ros2 run image_tools showimage --ros-args -r image:=/realsense/robot/color/image_raw
```

---

### 문제 4. `serial_no` 관련 type error 발생

`serial_no`는 문자열로 넘겨야 합니다.

권장 형태:

```bash
serial_no:="'243622073271'"
```

또는:

```bash
serial_no:="'244222070489'"
```

따옴표가 빠지면 integer type 관련 문제가 발생할 수 있습니다.

---

## 11. 최종 요약

필수 패키지:

```bash
sudo apt install -y \
  ros-humble-librealsense2* \
  ros-humble-realsense2-* \
  ros-humble-image-tools \
  ros-humble-rqt-image-view
```

실행 순서:

```bash
rs_enum
rs_vr
rs_robot
rs_topics
rs_hz_vr
rs_hz_robot
rs_view_vr
rs_view_robot
```

Robot 카메라가 USB2.x에서 불안정하면:

```bash
rs_robot_fallback
```

가장 중요한 주의점:

- bash에서 `=` 앞뒤 공백 금지
- topic 경로 중간 공백 금지
- `serial_no`는 문자열로 전달
- USB2.x 카메라는 해상도/FPS를 낮추는 것이 안정적
- 카메라 2대는 가능하면 서로 다른 USB controller 또는 USB3 포트에 연결
