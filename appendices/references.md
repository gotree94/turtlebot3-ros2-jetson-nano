# 참고 자료 (Jetson Nano 4GB)

> **RPi 버전 참고 자료:** `../../turtlebot3-ros2-curriculum/appendices/references.md` 참조
>
> 아래는 Jetson Nano 4GB 관련 추가 자료입니다.

---

## NVIDIA 공식 문서

| 자료 | 설명 | 링크 |
|------|------|------|
| Jetson Nano Developer Kit | 제품 페이지 | https://developer.nvidia.com/embedded/jetson-nano-developer-kit |
| Jetson Nano Module (양산용) | 모듈 데이터시트 | https://developer.nvidia.com/embedded/jetson-nano |
| JetPack 4.6 Archive | 최종 지원 JetPack | https://developer.nvidia.com/embedded/jetpack-sdk-461 |
| SDK Manager | 설치 도구 | https://developer.nvidia.com/sdk-manager |
| Jetson Download Center | 이미지 다운로드 | https://developer.nvidia.com/embedded/downloads |
| L4T Documentation | Linux for Tegra 문서 | https://docs.nvidia.com/jetson/l4t/ |

## Jetson Nano 커뮤니티 리소스

| 자료 | 설명 | 링크 |
|------|------|------|
| JetsonHacks | 튜토리얼, USB 부팅 가이드 | https://jetsonhacks.com/ |
| JetsonHacksNano GitHub | rootOnUSB, bootFromUSB 스크립트 | https://github.com/JetsonHacksNano |
| NVIDIA Developer Forums | Jetson Nano 포럼 | https://forums.developer.nvidia.com/c/jetson-nano/ |
| jetson-stats (jtop) | 시스템 모니터링 도구 | https://github.com/rbonghi/jetson_stats |
| Jetson Zoo | 추가 패키지 목록 | https://elinux.org/Jetson_Zoo |

## Docker + ROS2

| 자료 | 설명 | 링크 |
|------|------|------|
| dusty-nv jetson-containers | L4T 기반 Docker 이미지 | https://github.com/dusty-nv/jetson-containers |
| ROS2 Humble Docker (L4T) | ROS2 Humble for JetPack 4 | https://github.com/dusty-nv/jetson-containers/tree/master/packages/ros |
| CollaborativeRoboticsLab | Jetson Nano ROS2 설치 가이드 | https://github.com/AIResearchLab/JetsonNano-ROS2 |
| ROS2 Humble Install Guide | 공식 설치 문서 | https://docs.ros.org/en/humble/Installation.html |

## CUDA / AI / 컴퓨터 비전

| 자료 | 설명 | 링크 |
|------|------|------|
| CUDA 10.2 Documentation | Compute Capability 5.3 | https://docs.nvidia.com/cuda/archive/10.2/ |
| TensorRT 8.x | 추론 최적화 | https://developer.nvidia.com/tensorrt |
| jetson-inference | 추론 라이브러리 | https://github.com/dusty-nv/jetson-inference |
| jetbot | NVIDIA JetBot 프로젝트 | https://github.com/NVIDIA-AI-IOT/jetbot |
| CuPy (CUDA 10.2) | GPU 배열 연산 | `pip3 install cupy-cuda102` |

## 교육 자료

| 자료 | 설명 |
|------|------|
| NVIDIA DLI | https://www.nvidia.com/dli/ — Jetson 및 딥러닝 강좌 |
| Hello AI World | https://github.com/dusty-nv/jetson-inference |
| JetBot AI Kit | https://jetbot.org/ — Jetson Nano 기반 AI 로봇 |

## 스토리지 관련

| 자료 | 설명 | 링크 |
|------|------|------|
| rootOnUSB | USB 부팅 스크립트 | https://github.com/JetsonHacksNano/rootOnUSB |
| bootFromUSB | JetPack 4.5+ USB 부팅 | https://github.com/JetsonHacksNano/bootFromUSB |
| Jetson Nano USB Boot Guide | 가이드 | https://jetsonhacks.com/2021/03/10/jetson-nano-boot-from-usb/ |

## 성능 참고

| 항목 | Jetson Nano 4GB |
|------|----------------|
| GPU | NVIDIA Maxwell 128 CUDA cores @921MHz |
| CPU | Quad ARM Cortex-A57 @1.43GHz |
| RAM | 4GB 64-bit LPDDR4 25.6 GB/s |
| 스토리지 | microSD (기본) / USB SSD (권장) |
| AI 성능 | 472 GFLOPS (FP16) |
| 전력 | 5W ~ 10W |
| JetPack | 4.6.1 (최종 지원) |
| CUDA | 10.2 |
| ROS2 Humble | Docker 기반 |
| YOLO | YOLOv2 Tiny (~2-3 FPS, 128x128) |
| SLAM/Nav2 | Remote PC 필요 |
