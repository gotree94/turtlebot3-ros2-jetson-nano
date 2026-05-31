# Week 1 — 환경 구축 & ROS2 기초 (Jetson Nano 4GB)

> **목표:** Jetson Nano 4GB (SD 카드 모듈, eMMC 없음)에 JetPack 4.6.1 + Docker + ROS2 Humble을 설치하고 ROS2 개념을 마스터한다.

---

## 📅 주간 일정

| 일차 | 주제 | 핵심 내용 |
|------|------|----------|
| Day 1 | Jetson Nano OS & 스토리지 설정 | JetPack 4.6.1 설치, USB SSD 부팅, 개발용/양산용 차이 |
| Day 2 | Docker + ROS2 Humble 설치 | Docker 엔진, ROS2 Humble 컨테이너, 볼륨 마운트 |
| Day 3 | Remote PC 세팅 | Ubuntu 22.04 + ROS2 Humble Desktop |
| Day 4 | ROS2 핵심 개념 | Nodes, Topics, Services, Actions |
| Day 5 | 첫 ROS2 패키지 | Docker 내부에서 Publisher/Subscriber 작성 |
| Day 6 | 복습 & CUDA 설정 | CUDA 10.2 확인, GPU 모니터링, Docker GPU 패스스루 |
| Day 7 | 미니 프로젝트 | GPU/CPU 온도 모니터링 (Docker + 네이티브 혼합) |

---

## ⚠️ RPi / Orin Nano 버전과의 주요 차이점

| 항목 | RPi 4B | Jetson Orin Nano | **Jetson Nano 4GB (이 버전)** |
|------|--------|-------------------|-------------------------------|
| OS 설치 | Raspberry Pi Imager | SDK Manager | **SDK Manager (Host PC 필요)** |
| OS 버전 | Ubuntu 22.04 | Ubuntu 22.04 | **Ubuntu 18.04 (JetPack 4.6.1)** |
| ROS2 설치 | apt 네이티브 | apt 네이티브 | **Docker 컨테이너** |
| GPU | 없음 | Ampere 1024-core | **Maxwell 128-core** |
| CUDA | 없음 | 12.x | **10.2** |
| 스토리지 | microSD | NVMe SSD | **SD 카드 / USB SSD (eMMC 없음)** |
| RViz2 실행 | Remote PC | 온보드 | **Remote PC (RPi와 동일)** |
| 객체 탐지 | 불가능 | YOLOv8 실시간 | **YOLOv2 Tiny / 제한적** |
| USB SSD 부팅 | 해당 없음 | 해당 없음 | **필요 시 설정 가능** |

> 💡 **이 버전은 RPi 버전과 구조적으로 가장 유사합니다.**
> Remote PC가 필요하며, RViz2/SLAM/Nav2 시각화는 Remote PC에서 수행합니다.
> Jetson Nano의 장점은 **CUDA GPU**가 있다는 점으로, 제한된 AI/ML 실습이 가능합니다.
