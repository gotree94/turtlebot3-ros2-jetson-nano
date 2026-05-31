# 문제 해결 가이드 (Jetson Nano 4GB)

> **RPi 버전의 일반 문제 해결:** `../../turtlebot3-ros2-curriculum/appendices/troubleshooting.md` 참조
>
> 아래는 Jetson Nano 4GB 특화 문제입니다.

---

## 1. JetPack / SDK Manager 문제

### SDK Manager가 Jetson Nano를 찾지 못함

```bash
# 리커버리 모드 확인
lsusb | grep -i nvidia
# "NVidia Corp. APX"가 표시되어야 함

# 표시되지 않으면:
# 1. 전원 완전 차단 → 재시도
# 2. 다른 Micro-USB 케이블 사용 (데이터 전송 지원 케이블)
# 3. 다른 USB 포트 시도 (USB 3.0 권장)
```

### JetPack 4.6.1 다운로드 실패

```bash
# SDK Manager에서 오프라인 다운로드 옵션 선택
# 또는 NVIDIA Embedded Download Center에서 직접 다운로드
# https://developer.nvidia.com/embedded/downloads

# 수동 다운로드 후 SDK Manager → "Manual Setup" 선택
```

---

## 2. USB SSD 부팅 문제

### USB SSD 부팅 안 됨 (검은 화면)

```bash
# 1. QSPI-NOR 확인: JetPack 4.5+에서만 USB 부팅 가능
# 2. SD 카드를 넣은 상태에서 부팅 시도
# 3. QSPI-NOR 재플래싱:
#    SDK Manager 재실행 → "Force Recovery" 모드
# 4. extlinux.conf 확인:
#    root=/dev/sda1 (또는 PARTUUID)
```

### USB SSD 인식은 되나 부팅 실패

```bash
# PARTUUID 사용 (더 안정적)
sudo blkid /dev/sda1
# /dev/sda1: UUID="..." PARTUUID="abcdef12-01"

# extlinux.conf에서 root=PARTUUID=abcdef12-01 사용
```

### USB SSD 속도 저하

```bash
# USB 3.0 포트 사용 확인 (파란색 포트)
lsusb -t | grep 3.0

# SD 카드와 USB SSD 동시 사용 시 USB 대역폭 경합
# 가능하면 SD 카드를 제거하고 USB SSD만 사용
```

---

## 3. Docker 문제

### Docker 설치 실패

```bash
# arm64 저장소 확인
dpkg --print-architecture  # arm64

# Docker 저장소 설정 확인
cat /etc/apt/sources.list.d/docker.list
# deb [arch=arm64] https://download.docker.com/linux/ubuntu bionic stable
```

### NVIDIA Container Toolkit 오류

```bash
# GPU 패스스루 테스트
docker run --rm --gpus all nvidia/cuda:10.2-base nvidia-smi

# 오류 시:
# 1. nvidia-container-toolkit 재설치
sudo apt install --reinstall nvidia-container-toolkit
sudo systemctl restart docker

# 2. NVIDIA 드라이버 확인
ls /dev/nvidia*
# /dev/nvidia0, /dev/nvidiactl 등이 있어야 함
```

### Docker pull 매우 느림

```bash
# 이미지 다운로드 속도 개선
# 1. WiFi 대신 이더넷 사용
# 2. Docker mirror 설정
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": ["https://mirror.gcr.io"]
}
EOF
sudo systemctl restart docker

# 3. 대역폭 확인
iperf3 -c [server_ip]
```

### Docker 컨테이너 메모리 부족

```bash
# 컨테이너 메모리 제한 설정
docker run -itd --memory="2g" --memory-swap="3g" ...

# 현재 메모리 사용량
docker stats ros2_humble --no-stream
```

---

## 4. ROS2 / TurtleBot3 문제

### Docker 내부에서 Bringup 실패

```bash
# USB 장치 접근 확인
ls -l /dev/ttyACM0 /dev/ttyUSB0

# Docker 실행 시 권한 필요:
docker run -itd \
  --privileged \
  -v /dev:/dev \
  ...
```

### colcon build 메모리 부족

```bash
# 병렬 작업 수 제한
colcon build --parallel-workers 1

# Swap 메모리 설정 (SD 카드 수명 단축 주의)
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### ROS_DOMAIN_ID 불일치

```bash
# 모든 시스템(Jetson, Remote PC, Docker)에서 동일한 ID 확인
echo $ROS_DOMAIN_ID
# 모두 30으로 설정
```

### RViz2가 너무 느림 (Jetson Nano에서 직접 실행 시)

```bash
# Jetson Nano에서는 RViz2를 실행하지 말고 Remote PC에서 실행
# 강제로 실행해야 한다면:
rviz2 --display-config minimal.rviz  # 경량 설정
# PointCloud2, Grid 표시 제거
# LaserScan만 표시
```

---

## 5. 전원 / 온도 문제

### 전원 불안정 (갑자기 재부팅)

```bash
# 원인: 전력 부족
# 해결:
# 1. DC 배럴잭 5V 4A 사용 (Micro-USB 아님!)
# 2. J48 점퍼 확인 (DC 전원 활성화)
# 3. USB SSD 전력 소비 확인 (버스 파워드 vs 자체 전원)
# 4. 전원 어댑터 용량 확인 (최소 20W, 5V 4A)
```

### 과열로 쓰로틀링

```bash
# 온도 확인
cat /sys/devices/virtual/thermal/thermal_zone*/temp
# 85°C 이상이면 쓰로틀링 발생

# 해결:
# - 쿨링팬 연결 확인
# - 10W 모드 → 5W 모드 전환
sudo nvpmodel -m 1  # 5W 모드
# - 방열판 재부착 (써멀 패드 확인)
```

---

## 6. 저장 공간 문제

### eMMC 없음 → SD 카드/USB SSD 공간 부족

```bash
# 디스크 사용량 확인
df -h

# 불필요 파일 정리
sudo apt autoremove && sudo apt autoclean
sudo journalctl --vacuum-size=200M
docker system prune -a

# Docker 이미지를 USB SSD로 이동 (선택)
# /var/lib/docker 심볼릭 링크 변경
```

### Docker 이미지 용량 큼

```bash
# 사용 중인 이미지 확인
docker images

# 불필요 이미지/컨테이너 정리
docker system prune -a --volumes

# 권장: 최소 이미지만 유지
# dustynv/ros:humble-ros-base-l4t-r32.7.1 (~2GB)
```

---

## 7. 카메라 문제

### USB 카메라 미인식

```bash
# 장치 확인
ls /dev/video*

# 권한
sudo usermod -aG video $USER

# 테스트 (Docker 내부)
docker exec -it ros2_humble bash
ros2 run v4l2_camera v4l2_camera_node
```

### 카메라 노드 실행 안 됨 (Docker)

```bash
# Docker에 video 장치 마운트 확인
docker run -itd \
  --device /dev/video0:/dev/video0 \
  ...
```

---

## 문제 해결이 안 될 때

1. **NVIDIA Jetson Nano Forums**: https://forums.developer.nvidia.com/c/agx-autonomous-machines/jetson-embedded-systems/jetson-nano/
2. **JetsonHacks**: https://jetsonhacks.com/ (튜토리얼 및 팁)
3. **Jetson Zoo**: https://elinux.org/Jetson_Zoo
4. **dusty-nv jetson-containers**: https://github.com/dusty-nv/jetson-containers
5. **ROS2 Humble Docker Issue**: NVIDIA Developer Forum에서 "ROS2 Humble Nano" 검색
