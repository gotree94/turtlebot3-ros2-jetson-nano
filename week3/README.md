# Week 3 — SLAM, Navigation & 최종 프로젝트 (Jetson Nano 4GB)

> **목표:** SLAM으로 환경 지도를 생성하고, Navigation2로 자율주행하며,
> Jetson Nano의 제한된 GPU를 활용한 경량 AI 최종 프로젝트를 완성한다.

---

## 📅 주간 일정

| 일차 | 주제 | 실행 위치 |
|------|------|----------|
| Day 15 | SLAM 이론 | - |
| Day 16 | SLAM 매핑 | **Remote PC** (slam_toolbox) |
| Day 17 | Navigation2 개념 | - |
| Day 18 | Nav2 자율주행 | **Remote PC** (Nav2 시각화) |
| Day 19 | 최종 프로젝트 기획 | Jetson Nano + Remote PC |
| Day 20 | 최종 프로젝트 구현 | Jetson Nano (Docker) |
| Day 21 | 최종 프로젝트 완성 | 전체 시스템 |

---

## ⚠️ Jetson Nano vs Orin Nano Week 3 차이점

| 항목 | Jetson Orin Nano | **Jetson Nano 4GB** |
|------|------------------|---------------------|
| SLAM 실행 위치 | 온보드 가능 | **Remote PC 필요** |
| Nav2 실행 위치 | 온보드 가능 | **Remote PC 필요** |
| RViz2 실행 | 온보드 GPU 가속 | **Remote PC 필수** |
| AI 객체 탐지 | YOLOv8 실시간 (60FPS) | **YOLOv2 Tiny / 경량 모델 (~5FPS)** |
| 최종 프로젝트 | AI 기반 자율 순찰 | **제한된 AI 순찰** (간단한 객체 감지) |

> 💡 **Jetson Nano의 Week 3은 RPi 버전과 매우 유사합니다.**
> SLAM과 Nav2 시각화는 Remote PC에서 수행하며,
> Jetson Nano는 제한된 GPU를 활용한 기초 AI 실습에 집중합니다.
