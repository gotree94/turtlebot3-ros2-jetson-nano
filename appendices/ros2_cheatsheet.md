# ROS2 Humble Cheatsheet (Jetson Nano 4GB)

> **RPi 버전과 완전 동일합니다.** 아래는 Jetson Nano에 특화된 명령어만 추가합니다.

---

## Jetson Nano 특화 명령어

### 시스템 정보

```bash
# Jetson 모델 확인
cat /proc/device-tree/model

# CUDA 버전
nvcc --version

# GPU 정보 (Maxwell 128-core)
/usr/local/cuda-10.2/samples/bin/aarch64/linux/release/deviceQuery

# 전원 모드
sudo nvpmodel -q

# 실시간 모니터링
jtop

# 온도
cat /sys/devices/virtual/thermal/thermal_zone*/temp

# Docker 컨테이너 접속
docker exec -it ros2_humble bash
```

### Docker + ROS2

```bash
# ROS2 명령어 Docker alias 사용
ros2 topic list
ros2 run demo_nodes_cpp talker

# 또는 직접 접속 후 실행
docker exec -it ros2_humble bash
source /opt/ros/humble/setup.bash
source /ros2_ws/install/setup.bash
```

> **전체 ROS2 명령어:** `../../turtlebot3-ros2-curriculum/appendices/ros2_cheatsheet.md` 참조
>
> 포함 내용: nodes, topics, services, actions, bags, tf2, rqt, colcon 완전 동일
