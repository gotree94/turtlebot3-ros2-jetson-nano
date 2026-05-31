# Day 8 — TurtleBot3 SBC 패키지 설치 (Jetson Nano)

> **목표:** Jetson Nano에 TurtleBot3 관련 ROS2 패키지를 설치한다.
> 네이티브와 Docker 컨테이너를 혼용하여 최적의 환경을 구성한다.

---

## 8.1 패키지 설치 전략

Jetson Nano (JetPack 4.6.1, Ubuntu 18.04)에서는 ROS2 Humble apt 패키지가 없습니다.
따라서 TurtleBot3 패키지를 다음 두 방식으로 설치합니다:

```
┌─────────────────────────────────────────────────────┐
│  Jetson Nano                                        │
│                                                     │
│  네이티브 (Ubuntu 18.04)                             │
│  ├── TurtleBot3 브링업 관련                          │
│  │   - turtlebot3_bringup (소스 빌드 또는 바이너리)   │
│  │   - hls_lfcd_lds_driver (LIDAR 드라이버)          │
│  │   - DYNAMIXEL SDK (OpenCR 통신)                   │
│  └── 이유: USB/시리얼 직접 접근 필요                   │
│                                                     │
│  Docker (ROS2 Humble)                                │
│  ├── ROS2 패키지 개발 관련                            │
│  │   - turtlebot3_msgs (소스 빌드)                   │
│  │   - turtlebot3_interfaces (소스 빌드)             │
│  │   - 사용자 패키지 (pub/sub, PID, 장애물 회피)      │
│  └── 이유: ROS2 Humble 네이티브 환경                  │
│                                                     │
│  Remote PC (Ubuntu 22.04)                            │
│  ├── RViz2, rqt 관련                                 │
│  │   - turtlebot3_teleop (키보드 텔레옵)             │
│  │   - turtlebot3_navigation2 (Nav2 시각화)          │
│  │   - rviz2, rqt                                    │
│  └── 이유: GPU 가속 RViz2 렌더링                      │
└─────────────────────────────────────────────────────┘
```

---

## 8.2 네이티브: TurtleBot3 Bringup 패키지 설치

Jetson Nano 네이티브(Ubuntu 18.04)에서 TurtleBot3 하드웨어를 직접 제어합니다.

```bash
# ROS1 Melodic (L4T 기본)이 아닌 ROS2 Humble을 사용하므로
# turtlebot3 패키지는 GitHub 소스로 직접 빌드

# 1. 의존성 설치
sudo apt update
sudo apt install -y \
  python3-pip \
  python3-serial \
  python3-numpy \
  python3-empy

# 2. ROS2 Humble 패키지 저장소 (Jetson Nano 네이티브)
# 실제 ROS2 노드는 Docker에서 실행하지만,
# OpenCR/LIDAR 펌웨어와 관련된 도구는 네이티브에 설치

# 3. Docker 컨테이너 내부에서 TurtleBot3 패키지 설치
docker exec -it ros2_humble bash

# (컨테이너 내부)
source /opt/ros/humble/setup.bash
cd /ros2_ws/src

# 4. TurtleBot3 관련 패키지 클론
git clone -b humble-devel https://github.com/ROBOTIS-GIT/turtlebot3_msgs.git
git clone -b humble-devel https://github.com/ROBOTIS-GIT/turtlebot3.git
git clone -b ros2-devel https://github.com/ROBOTIS-GIT/hls_lfcd_lds_driver.git

# 5. 의존성 설치
cd ..
rosdep update
rosdep install -i --from-path src --rosdistro humble -y

# 6. 빌드
colcon build --symlink-install

# 7. 환경 설정
source install/setup.bash
echo 'source /ros2_ws/install/setup.bash' >> ~/.bashrc
```

> ⚠️ **빌드 시간:**
> - USB SSD: ~10분
> - SD 카드: ~40~60분

---

## 8.3 OpenCR 펌웨어 (네이티브)

OpenCR 보드에 TurtleBot3 Burger 펌웨어를 업로드합니다.

```bash
# 네이티브 Jetson Nano에서 실행
# (USB 직접 접근 필요)

# 1. OpenCR USB 연결 확인
ls /dev/ttyACM0
ls -l /dev/ttyACM0

# 2. Docker 컨테이너가 USB 접근 가능한지 확인
ls -l /ros2_ws/src/turtlebot3/turtlebot3_bringup/scripts/
# 또는 네이티브에서 OpenCR 툴체인 설치

# 3. OpenCR 펌웨어 업로드 스크립트 (Docker 내부)
docker exec -it ros2_humble bash
source /opt/ros/humble/setup.bash
source /ros2_ws/install/setup.bash

export OPENCR_PORT=/dev/ttyACM0
export OPENCR_MODEL=burger

# 펌웨어 업로드
ros2 run turtlebot3_bringup opencr_update.py \
  --model ${OPENCR_MODEL} \
  --port ${OPENCR_PORT}
```

---

## 8.4 환경 변수 설정

```bash
# Docker 컨테이너 내부 ~/.bashrc
cat >> ~/.bashrc << 'EOF'
source /opt/ros/humble/setup.bash
source /ros2_ws/install/setup.bash
export ROS_DOMAIN_ID=30
export TURTLEBOT3_MODEL=burger
export LDS_MODEL=LDS-01
EOF
```

---

## 8.5 네이티브 Bringup (권장)

Docker 대신 네이티브에서 bringup을 실행하는 이유:
1. USB/시리얼 포트 접근이 더 안정적
2. 실시간 성능이 더 나음
3. 전원 관리(power mode) 직접 제어 가능

```bash
# (네이티브 Jetson Nano)
export TURTLEBOT3_MODEL=burger
export ROS_DOMAIN_ID=30

# LIDAR 실행 (네이티브)
ros2 launch turtlebot3_bringup robot.launch.py

# 또는 LIDAR 드라이버만 따로 (Docker)
docker exec -it ros2_humble bash
ros2 launch hls_lfcd_lds_driver hlds_laser.launch.py
```

> 💡 **실제 운영 시 팁:** `robot.launch.py`를 Docker 컨테이너 내부에서 실행할 수도 있습니다.
> 다만 USB 접근을 위해 `-v /dev:/dev --privileged` 옵션이 필요합니다.

---

## 8.6 설치 확인

```bash
# Docker 컨테이너에서 확인
docker exec -it ros2_humble bash
source /opt/ros/humble/setup.bash
source /ros2_ws/install/setup.bash

# 패키지 확인
ros2 pkg list | grep turtlebot3
# turtlebot3_bringup
# turtlebot3_msgs
# turtlebot3

# 패키지 실행 테스트 (오류 없이 도움말 출력)
ros2 launch turtlebot3_bringup robot.launch.py -s
```

---

## 📝 연습 문제

1. Docker 컨테이너에서 빌드한 turtlebot3_msgs 패키지 목록을 확인하세요
2. 네이티브 bringup과 Docker bringup의 장단점을 비교하세요
3. USB SSD와 SD 카드에서 colcon build 시간을 측정하고 비교하세요
4. OpenCR 펌웨어 버전을 확인하는 명령어를 찾아보세요

---

## ⚠️ 트러블슈팅

| 문제 | 해결 방법 |
|------|----------|
| `ttyACM0` 권한 없음 | `sudo usermod -aG dialout $USER`, 로그아웃 후 재접속 |
| colcon build 실패 | `rosdep update` 후 재시도, 인터넷 연결 확인 |
| turtlebot3 브랜치 없음 | `--branch humble-devel` 확인, `ros2-devel`로 대체 |
| Docker USB 접근 안 됨 | `--privileged` 옵션 확인, `-v /dev:/dev` 볼륨 마운트 |
| 빌드 너무 느림 | USB SSD 사용으로 전환 권장 |
