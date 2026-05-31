# Day 6 — 복습 & CUDA 설정 확인

> **목표:** Week 1 내용을 복습하고 Jetson Nano의 CUDA 10.2 GPU 개발 환경을 확인한다.

---

## 6.1 CUDA 10.2 확인

```bash
# CUDA 버전
nvcc --version

# 출력 예시:
# nvcc: NVIDIA (R) Cuda compiler driver
# Copyright (c) 2005-2019 NVIDIA Corporation
# Built on ...
# Cuda compilation tools, release 10.2, V10.2.89

# CUDA 설치 경로
ls /usr/local/cuda
ls /usr/local/cuda-10.2

# GPU 정보
nvidia-smi  # Jetson Nano에서는 제한적 정보
# 대신 tegrastats 사용
sudo tegrastats

# GPU 아키텍처
cat /proc/device-tree/model
# Jetson Nano Developer Kit
```

---

## 6.2 Docker 내 CUDA 확인

```bash
# 컨테이너에서 CUDA 사용 가능 확인
docker exec ros2_humble nvidia-smi

# 또는
docker run --rm --gpus all \
  dustynv/l4t-cuda:r32.7.1 \
  nvidia-smi
```

---

## 6.3 시스템 모니터링 도구

```bash
# jtop (jetson-stats) — Jetson 전용 모니터링
sudo pip3 install jetson-stats
jtop
# CPU/GPU 사용률, 온도, 전력, 메모리 실시간 표시

# tegrastats — 경량 CLI 모니터링
sudo tegrastats
# RAM 3941/3941MB (lfb 1204/1204MB) CPU [12%@1479,18%@1479,...]
# GR3D_FREQ 76%@921 EMC_FREQ 88%@1600
# CPU@55.5 GPU@53 ADC@48.5 Tboard@41 Tdiode@61.5

# nvtop — GPU 프로세스 모니터링
sudo apt install -y nvtop
nvtop
```

---

## 6.4 CUDA 프로그래밍 기초 (선택)

Jetson Nano의 128 CUDA 코어를 활용한 간단한 GPU 프로그래밍:

```bash
# CUDA 샘플 설치
sudo apt install -y cuda-samples-10-2

# deviceQuery 실행
/usr/local/cuda-10.2/samples/bin/aarch64/linux/release/deviceQuery

# 출력 예시:
# CUDA Device Query (Runtime API) version (CUDART static linking)
# Detected 1 CUDA Capable device(s)
# Device 0: "NVIDIA Tegra X1"
#   CUDA Capability Major/Minor version number: 5.3
#   Total amount of global memory: 3964 MBytes
#   (1) Multiprocessors, (128) CUDA Cores/MP: 128 CUDA Cores
```

---

## 6.5 Week 1 복습 퀴즈

1. Jetson Nano SD 카드 모듈과 eMMC 모듈의 차이는?
2. ROS2 Humble을 Docker로 설치하는 이유는?
3. USB SSD 부팅의 장점은?
4. jetson-stats로 확인할 수 있는 정보 3가지는?
5. J48 점퍼의 역할은?
6. Remote PC가 필수인 이유는? (Jetson Nano 한계)

---

## 📝 연습 문제

1. CUDA deviceQuery를 실행하고 출력된 GPU 정보를 기록하세요
2. `jtop`에서 GPU 사용률이 0%인 상태와 부하 상태를 각각 캡쳐하세요
3. `sudo tegrastats`를 10초 간격으로 실행하고 온도 변화를 관찰하세요
4. CUDA 10.2에서 지원하는 Compute Capability (5.3)의 의미를 조사하세요
5. Docker 컨테이너에서 CUDA가 정상 동작하는지 확인하는 명령어를 작성하세요
