# Day 2 — Docker + ROS2 Humble 설치

> **목표:** Jetson Nano (Ubuntu 18.04, JetPack 4.6.1)에 Docker Engine을 설치하고,
> ROS2 Humble 컨테이너를 실행하여 ROS2 환경을 구축한다.

---

## 2.1 왜 Docker인가?

Jetson Nano는 **JetPack 4.6.1 (Ubuntu 18.04)** 까지만 공식 지원합니다.
ROS2 Humble은 **Ubuntu 22.04 (Jammy)** 를 Tier 1로 요구하므로,
Ubuntu 18.04에서 apt로 직접 설치할 수 없습니다.

```
해결 방법:

┌─────────────────────────────────────────────────────┐
│  Jetson Nano (Ubuntu 18.04 + JetPack 4.6.1)         │
│                                                      │
│  ┌────────────────────────────────────────────────┐  │
│  │  Docker Container                              │  │
│  │                                                │  │
│  │  ┌──────────────────────────────────────────┐  │  │
│  │  │  ROS2 Humble (L4T r32.7.1 기반)           │  │  │
│  │  │  - Ubuntu 20.04 (컨테이너 내부)           │  │  │
│  │  │  - ROS2 Humble ROS-Base                    │  │  │
│  │  │  - CUDA 10.2 (GPU 패스스루)               │  │  │
│  │  │  - colcon workspace 볼륨 마운트            │  │  │
│  │  └──────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────┘  │
│                                                      │
│  ┌────────────────────────────────────────────────┐  │
│  │  Host (네이티브)                                │  │
│  │  - JetPack 4.6.1 (CUDA 10.2, L4T 드라이버)     │  │
│  │  - TurtleBot3 bringup (네이티브 권장)           │  │
│  │  - jtop, tegrastats (시스템 모니터링)           │  │
│  └────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

**권장 아키텍처:**
- **네이티브**: 시스템 모니터링, TurtleBot3 bringup (직접 하드웨어 접근)
- **Docker**: ROS2 패키지 개발, 빌드(colcon), ROS2 노드 실행
- **Remote PC**: RViz2, rqt 시각화

---

## 2.2 Docker Engine 설치

```bash
# 기존 패키지 제거
sudo apt remove docker docker-engine docker.io containerd runc

# 의존성 설치
sudo apt update
sudo apt install -y \
  apt-transport-https \
  ca-certificates \
  software-properties-common

# Docker GPG 키 추가
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo apt-key add -

# Docker 저장소 추가
sudo add-apt-repository \
  "deb [arch=arm64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable"

# Docker Engine 설치
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# 현재 사용자를 docker 그룹에 추가
sudo usermod -aG docker $USER
newgrp docker

# Docker 동작 확인
docker run hello-world
# 또는 (arm64용):
docker run arm64v8/hello-world

# Docker 버전 확인
docker --version
```

---

## 2.3 NVIDIA Container Toolkit (GPU 패스스루)

Jetson Nano의 CUDA GPU를 Docker 컨테이너 내부에서 사용하기 위해 필요합니다.

```bash
# Docker에서 NVIDIA GPU 사용 설정
# JetPack 4.6.1에는 nvidia-docker2가 기본 패키지로 포함되어 있지 않음

# NVIDIA Container Toolkit 저장소 추가
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list

# nvidia-docker2 설치
sudo apt update
sudo apt install -y nvidia-container-toolkit

# Docker 재시작
sudo systemctl restart docker

# GPU 패스스루 확인
docker run --rm --gpus all nvidia/cuda:10.2-base nvidia-smi
```

> 참고: `nvidia/cuda:10.2-base` 이미지는 기본적으로 x86_64용입니다.
> Jetson Nano (aarch64)에서는 다음 명령어로 대체 확인:
> ```bash
> docker run --rm --gpus all \
>   dustynv/l4t-cuda:r32.7.1 \
>   nvidia-smi
> ```

---

## 2.4 ROS2 Humble Docker 이미지

dusty-nv의 jetson-containers에서 제공하는 ROS2 Humble 이미지를 사용합니다.

### 이미지 종류

| 이미지 | 크기 | 내용 | 사용 |
|--------|------|------|------|
| `dustynv/ros:humble-ros-base-l4t-r32.7.1` | ~2GB | ROS2 Base (rclcpp, rclpy) | **ROS2 개발용** |
| `dustynv/ros:humble-desktop-l4t-r32.7.1` | ~4GB | + RViz2, rqt | RViz2는 Remote PC에서 실행 |

### ROS2 Humble 컨테이너 실행

```bash
# 1. 워크스페이스 디렉토리 생성 (네이티브)
mkdir -p ~/ros2_ws/src

# 2. ROS2 Humble Base 이미지 pull
docker pull dustynv/ros:humble-ros-base-l4t-r32.7.1

# 3. 컨테이너 실행 (백그라운드)
docker run -itd \
  --name ros2_humble \
  --gpus all \
  --runtime nvidia \
  -e DISPLAY=$DISPLAY \
  -e ROS_DOMAIN_ID=30 \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -v ~/ros2_ws:/ros2_ws \
  -v /dev:/dev \
  --network host \
  --privileged \
  dustynv/ros:humble-ros-base-l4t-r32.7.1

# 4. 컨테이너 접속
docker exec -it ros2_humble bash
```

### Docker 볼륨 구조

```
Jetson Nano Host                        Docker Container
─────────────────                      ────────────────
~/ros2_ws/  ──────── 볼륨 마운트 ─────► /ros2_ws/
  ├── src/                               ├── src/
  ├── build/                             ├── build/
  ├── install/                           ├── install/
  └── log/                               └── log/
```

> 💡 **볼륨 마운트의 장점:**
> - 네이티브에서 소스 코드 편집 → 컨테이너에서 빌드/실행
> - 컨테이너를 삭제해도 코드는 유지
> - Remote PC와 동기화 가능

---

## 2.5 ROS2 Humble 동작 확인

```bash
# 컨테이너 내부에서 (docker exec -it ros2_humble bash)
source /opt/ros/humble/setup.bash

# ROS2 환경 확인
ros2 --version
# ROS 2 Humble Hawksbill

# 토픽 리스트 확인
ros2 topic list

# talker/listener 예제 실행 (터미널 2개 필요)
# 터미널 1:
ros2 run demo_nodes_cpp talker

# 터미널 2 (별도 창):
docker exec -it ros2_humble bash
ros2 run demo_nodes_py listener
```

---

## 2.6 개발 편의: 별칭(alias) 설정

Jetson Nano 네이티브에서 ROS2 명령어를 바로 사용할 수 있도록 설정:

```bash
# ~/.bashrc에 추가
cat >> ~/.bashrc << 'EOF'

# ROS2 Docker alias
alias ros2='docker exec -it ros2_humble bash -c "source /opt/ros/humble/setup.bash && ros2 $*"'
alias colcon='docker exec -it ros2_humble bash -c "source /opt/ros/humble/setup.bash && cd /ros2_ws && colcon $*"'
alias r2bash='docker exec -it ros2_humble bash'
alias r2src='docker exec -it ros2_humble bash -c "source /opt/ros/humble/setup.bash && source /ros2_ws/install/setup.bash && exec bash"'
EOF
source ~/.bashrc
```

---

## 2.7 GUI 응용 프로그램 (X11 Forwarding)

Jetson Nano에서 GUI 앱을 실행하려면 X11 forwarding이 필요합니다.

```bash
# 네이티브에서 xhost 허용
xhost +local:docker

# 컨테이너에 DISPLAY 환경변수 전달 확인
# (위 docker run 명령어에 이미 포함됨)
echo $DISPLAY
# :0 (또는 :1)
```

> ⚠️ **참고:** Jetson Nano의 Maxwell GPU는 RViz2를 실행하기에는 성능이 부족합니다.
> (특히 3D PointCloud 렌더링) **RViz2는 Remote PC에서 실행**하는 것을 권장합니다.
> 간단한 rqt, rviz2 경량 사용에만 제한적으로 활용하세요.

---

## 2.8 컨테이너 관리

```bash
# 컨테이너 중지
docker stop ros2_humble

# 컨테이너 시작
docker start ros2_humble

# 컨테이너 접속
docker exec -it ros2_humble bash

# 로그 확인
docker logs ros2_humble

# 컨테이너 삭제 (워크스페이스는 유지)
docker rm ros2_humble

# 이미지 목록
docker images

# 불필요 이미지 정리
docker system prune
```

---

## 2.9 전체 설치 확인

```bash
echo "===== 시스템 정보 ====="
cat /proc/device-tree/model
uname -a

echo "===== CUDA ====="
nvcc --version

echo "===== Docker ====="
docker --version
docker ps | grep ros2_humble

echo "===== Docker GPU ====="
docker exec ros2_humble bash -c "ls /usr/local/cuda/bin/nvcc && /usr/local/cuda/bin/nvcc --version"

echo "===== ROS2 ====="
docker exec ros2_humble bash -c "source /opt/ros/humble/setup.bash && ros2 --version"

echo "===== 디스크 ====="
df -h
```

---

## 📝 연습 문제

1. Docker를 사용하는 이유를 ROS2 Humble과 JetPack 4.6.1의 관계로 설명하세요
2. Docker 볼륨 마운트가 ROS2 패키지 개발에 주는 이점은?
3. `docker exec`와 `docker run`의 차이점을 설명하세요
4. NVIDIA Container Toolkit 없이 Docker에서 CUDA를 사용할 수 없는 이유는?
5. ROS2 Humble talker/listener 예제를 실행하고 토픽 메시지를 확인하세요

---

## ⚠️ 트러블슈팅

| 문제 | 해결 방법 |
|------|----------|
| `docker: command not found` | Docker Engine 재설치, sudo usermod 후 로그아웃/로그인 |
| GPU 패스스루 안 됨 | `--gpus all` 확인, nvidia-container-toolkit 재설치 |
| 컨테이너 내 ros2 명령어 없음 | `source /opt/ros/humble/setup.bash` 실행 |
| 볼륨 마운트 안 됨 | Docker Desktop이 아니라 Docker Engine인지 확인 |
| X11 앱 실행 안 됨 | `xhost +local:docker`, DISPLAY 환경변수 확인 |
| Docker pull 매우 느림 | WiFi 대신 이더넷 사용, USB SSD 부팅인지 확인 |
