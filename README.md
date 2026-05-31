# 🐢 TurtleBot3 Burger + Jetson Nano + ROS2 Humble

> **Jetson Nano 4GB (eMMC 없음 / SD 카드 모듈)** 환경에서 ROS2 Humble을 구축하고,
> TurtleBot3 Burger를 제어하며 SLAM/네비게이션/기초 AI까지 마스터하는 3주 집중 코스

---

## 📋 개요

이 커리큘럼은 **TurtleBot3 Burger** 로봇에 **NVIDIA Jetson Nano 4GB**를 SBC로 탑재하고,<br>
**ROS2 Humble Hawksbill** (Docker 기반) 환경에서 최종 프로젝트까지 완성하는 것을 목표로 합니다.

| 항목 | 내용 |
|------|------|
| **총 기간** | 3주 (21일) |
| **난이도** | 초중급 (Linux 기초 지식 필요) |
| **ROS 버전** | ROS2 Humble Hawksbill (Docker) |
| **타겟 보드** | NVIDIA Jetson Nano 4GB (Maxwell GPU 128-core) |
| **SBC 모듈 유형** | SD 카드 모듈 (P3448-0000, eMMC 없음) |
| **OS** | Ubuntu 18.04 LTS aarch64 (JetPack 4.6.1) |
| **로봇** | TurtleBot3 Burger |
| **Remote PC** | **필수** (RViz2, SLAM, Nav2 시각화용) |

---

## 🧠 Jetson Nano 하드웨어 상세: 개발용 vs 양산용

Jetson Nano는 **두 가지 모듈 버전**이 존재하며, 각각 용도와 저장 장치가 다릅니다.

### 모듈 비교

| 항목 | 개발용 (Dev Kit) | 양산용 (Production Module) |
|------|------------------|---------------------------|
| **모델 번호** | P3448-0000 | P3448-0001 / P3448-0002 |
| **목적** | 학습, 프로토타이핑, 교육 | 상용 제품, 산업용 임베디드 |
| **저장 장치** | ❌ **eMMC 없음** (SD 카드 전용) | ✅ **16GB eMMC 5.1** (온보드) |
| **SD 카드 슬롯** | ✅ 있음 (모듈 자체에 있음) | ❌ 없음 |
| **부팅 방식** | microSD 카드 → USB SSD (QSPI-NOR) | eMMC 직접 부팅 |
| **가격** | ~$99 (Developer Kit) | ~$129 (모듈 단품) |
| **보증** | 1년 제한 보증 | 확장 보증, 산업용 온도 범위 |
| **캐리어 보드** | NVIDIA 참조 보드(P3449) 포함 | 미포함 (별도 제작 또는 구매) |
| **대상** | 메이커, 연구자, 학생 | 임베디드 설계자, 제품 개발사 |

### ⚠️ "eMMC 없는 버전"이란?

이 커리큘럼이 타겟으로 하는 **Jetson Nano SD 카드 모듈 (P3448-0000)**:

```
┌─────────────────────────────────────────────┐
│   Jetson Nano Module (P3448-0000)           │
│                                             │
│  • GPU: Maxwell 128-core                    │
│  • CPU: Cortex-A57 x4 @1.43GHz             │
│  • RAM: 4GB LPDDR4 (25.6 GB/s)            │
│  • eMMC: ❌ 없음                            │
│  • SD 카드 슬롯: ✅ 모듈 자체에 있음       │
│  • 부트로더: QSPI-NOR (SPI flash)          │
└─────────────────────────────────────────────┘
```

- **저장 장치가 전혀 없는** 순수 컴퓨팅 모듈입니다
- OS는 반드시 **microSD 카드** 또는 **USB SSD**에 설치해야 합니다
- Developer Kit에는 이 모듈 + 참조 캐리어 보드가 함께 들어있습니다

### 저장 장치 권장 사항

| 저장 장치 | 속도 (읽기) | 신뢰성 | ROS2 작업 적합성 | 비고 |
|----------|-----------|-------|----------------|------|
| microSD (Class 10 U3) | ~87 MB/s | ❌ 낮음 | ⚠️ 기본 부팅 가능 | ROS2 빌드 시 매우 느림 |
| USB 3.0 SSD | ~367 MB/s | ✅ 높음 | ✅ **적합** | **강력 권장** (4배 속도 향상) |
| USB 3.0 HDD | ~80 MB/s | ✅ 높음 | ⚠️ 보통 | 대용량 저장, 속도는 SD 수준 |
| SATA SSD + USB 변환 | ~350 MB/s | ✅ 높음 | ✅ 적합 | 경제적인 SSD 옵션 |

> 💡 **USB SSD 사용을 강력히 권장합니다.** SD 카드는 ROS2 패키지 빌드(colcon build) 시
> I/O 병목으로 수십 분이 소요될 수 있습니다. USB SSD 사용 시 5~10분으로 단축됩니다.

---

## 🛠 사전 준비물

### 하드웨어

| 항목 | 비고 |
|------|------|
| TurtleBot3 Burger | OpenCR 1.0, DYNAMIXEL XL430-W250 x2, 배터리 포함 |
| NVIDIA Jetson Nano Developer Kit (4GB) | SD 카드 모듈 버전 (eMMC 없음) |
| MicroSD 카드 | 64GB 이상, Class 10 U3 / A2 속도 권장 |
| USB 3.0 SSD (권장) | 128GB 이상 — OS 및 ROS2 워크스페이스용 |
| MicroSD 리더기 | OS 플래싱용 (호스트 PC) |
| DC 배럴잭 전원 어댑터 | 5V 4A (20W) — **필수** (Micro-USB만으로는 부족) |
| 점퍼 와이퍼 (J48) | DC 배럴잭 전원 활성화용 (초기 설정) |
| HDMI 모니터 + USB 키보드/마우스 | 초기 설정용 |
| Remote PC (노트북/데스크탑) | **필수** — Ubuntu 22.04 Desktop (RViz2, SLAM, Nav2 시각화) |
| WiFi 공유기 | 동일 네트워크 (5GHz 권장) |
| USB 카메라 (선택) | TurtleBot3 Burger 마운트 가능한 모델 |
| 방열판 + 쿨링팬 | Jetson Nano 발열 필수 대책 |

### 소프트웨어

| 항목 | 버전 |
|------|------|
| NVIDIA JetPack | 4.6.1 (Ubuntu 18.04 LTS aarch64) |
| SDK Manager | 최신 버전 (호스트 PC, Ubuntu 20.04/22.04 x86_64) |
| CUDA | 10.2 (JetPack 포함) |
| Docker Engine | 20.x+ (apt 공식 저장소) |
| ROS 2 (Docker) | Humble Hawksbill (`dustynv/ros:humble-ros-base-l4t-r32.7.1`) |
| Ubuntu Desktop (Remote PC) | 22.04 LTS amd64 |
| ROS 2 (Remote PC) | Humble Hawksbill Desktop (네이티브 apt 설치) |

---

## Jetson Nano vs 다른 플랫폼 비교

| 항목 | Jetson Nano 4GB | Raspberry Pi 4B (8GB) | Jetson Orin Nano (8GB) |
|------|----------------|----------------------|------------------------|
| **CPU** | Cortex-A57 x4 @1.43GHz | Cortex-A72 x4 @1.8GHz | Cortex-A78AE x6 @1.5GHz |
| **GPU** | NVIDIA Maxwell 128-core | ❌ 없음 | NVIDIA Ampere 1024-core |
| **AI 성능** | 472 GFLOPS (FP16) | ❌ 불가 | 40 TOPS (INT8) |
| **메모리** | 4GB LPDDR4 | 8GB LPDDR4 | 8GB/16GB LPDDR5 |
| **메모리 대역폭** | 25.6 GB/s | ~3 GB/s | 68 GB/s |
| **스토리지** | SD 카드 / USB SSD | microSD | NVMe SSD (M.2) |
| **JetPack / OS** | 4.6.1 (Ubuntu 18.04) | Ubuntu Server 22.04 | 6.x (Ubuntu 22.04) |
| **CUDA** | 10.2 | ❌ | 12.x |
| **ROS2 Humble** | Docker 기반 | 네이티브 apt | 네이티브 apt |
| **SLAM+Nav2** | Remote PC 필요 | Remote PC 필요 | ✅ 온보드 |
| **RViz2** | Remote PC 필요 | Remote PC 필요 | ✅ 온보드 GPU |
| **객체 탐지** | ⚠️ 제한적 (YOLOv2/v3 Tiny) | ❌ 불가 | ✅ 실시간 YOLOv8 |
| **전력** | 5W / 10W | 5~7W | 7~15W |
| **가격** | ~$99 | ~$75 | ~$499 |

---

## 📚 주차별 학습 로드맵

```
Week 1 ────┬── Day 1~2: JetPack 4.6.1 + Docker + ROS2 Humble 설치
           ├── Day 3~4: Remote PC 세팅 + ROS2 개념 (RPi와 동일)
           ├── Day 5~6: 첫 패키지 + CUDA 10.2 확인
           └── Day 7: [미니 프로젝트] GPU 온도 모니터링

Week 2 ────┬── Day 8~9: TurtleBot3 SBC + PC 패키지 (Docker 기반)
           ├── Day 10~11: Bringup + 센서 시각화 (Remote PC)
           ├── Day 12~13: PID 제어 + URDF (RPi와 동일)
           └── Day 14: [미니 프로젝트] LIDAR 장애물 회피

Week 3 ────┬── Day 15~16: SLAM 이론 + 실제 매핑 (Remote PC)
           ├── Day 17~18: Navigation2 + 자율주행 (Remote PC)
           └── Day 19~21: [최종 프로젝트] 제한된 AI 순찰 로봇
```

---

## 🔑 이 버전의 핵심 특징

1. **Docker 기반 ROS2 Humble**: JetPack 4.6.1(Ubuntu 18.04) 위에서 Docker 컨테이너로 ROS2 Humble 실행
2. **USB SSD 부팅 권장**: SD 카드 한계 극복을 위한 USB SSD 루트 파일시스템 구성
3. **CUDA 10.2 + Maxwell GPU**: 제한된 GPU 성능으로 기초 AI/ML 실습 가능
4. **Remote PC 필수**: RViz2, SLAM, Nav2 시각화는 Remote PC에서 수행 (RPi와 유사)
5. **제한된 객체 탐지**: YOLOv2 Tiny 또는 TensorRT 경량 모델로 기초 AI 순찰 구현
6. **eMMC 없음 대비**: SD 카드 모듈 전용 스토리지 설정 가이드 포함
7. **개발용/양산용 구분**: 모듈 타입별 차이점과 저장 장치 선택 가이드

---

## 📂 디렉토리 구조

```
turtlebot3-ros2-curriculum-jetsonano/
├── README.md
├── week1/
│   ├── README.md
│   ├── day01_jetson_nano_setup.md       ← Jetson Nano 전용 (개발용/양산용 설명)
│   ├── day02_docker_ros2_install.md      ← Docker + ROS2 Humble 설치
│   ├── day03_remote_pc_setup.md          ← RPi와 동일
│   ├── day04_ros2_concepts.md            ← RPi와 동일
│   ├── day05_first_package.md            ← RPi와 동일 (Docker 볼륨 마운트)
│   ├── day06_review_cuda_setup.md        ← CUDA 10.2 확인
│   └── day07_mini_project_gpu_temp.md    ← GPU 온도 모니터링
├── week2/
│   ├── README.md
│   ├── day08_turtlebot3_sbc_setup.md     ← Docker 기반 패키지 설치
│   ├── day09_remote_pc_tb3_pkg.md        ← RPi와 동일
│   ├── day10_bringup_teleop.md           ← RPi와 동일
│   ├── day11_sensor_visualization.md     ← RPi와 동일 (Remote PC)
│   ├── day12_pid_control_basics.md       ← RPi와 동일
│   ├── day13_rviz2_urdf.md              ← RPi와 동일
│   └── day14_mini_project_obstacle.md    ← RPi와 동일
├── week3/
│   ├── README.md
│   ├── day15_slam_theory.md              ← RPi와 동일
│   ├── day16_slam_mapping.md             ← RPi와 동일 (Remote PC)
│   ├── day17_navigation2_concepts.md     ← RPi와 동일
│   ├── day18_nav2_autonomous.md          ← RPi와 동일 (Remote PC)
│   ├── day19_final_project_plan.md       ← Nano AI 순찰 기획
│   ├── day20_final_project_impl.md       ← Nano 경량 AI 구현
│   └── day21_final_project_final.md      ← 최종 완성
├── exercises/
└── appendices/
```

---

## 🚀 빠른 시작

```bash
git clone https://github.com/your-username/turtlebot3-ros2-curriculum-jetsonano.git
cd turtlebot3-ros2-curriculum-jetsonano
# week1/day01부터 순서대로 진행
```

---

## 📝 라이선스

MIT License
