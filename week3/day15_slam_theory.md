# Day 15 — SLAM 이론

> **RPi 버전 Day 15와 내용이 동일합니다.**

---

## SLAM 실행 위치

Jetson Nano에서는 SLAM 알고리즘(slam_toolbox) 자체는 **Remote PC에서 실행**합니다.
(CPU 성능과 메모리(RAM 4GB)의 한계로 온보드 SLAM 실행 시 메모리 부족 위험)

```
┌─────────────────┐  /scan, /odom, /tf  ┌──────────────────┐
│  Jetson Nano    │ ◄──────────────────► │  Remote PC       │
│  (네이티브)      │                      │                  │
│                 │                      │  slam_toolbox    │
│  Bringup 실행   │                      │  RViz2           │
│  LIDAR+IMU 발행 │                      │  지도 저장        │
└─────────────────┘                      └──────────────────┘
```

> **전체 내용:** `../../turtlebot3-ros2-curriculum/week3/day15_slam_theory.md` 참조
>
> 이론 내용: SLAM 정의, EKF vs GraphSLAM, slam_toolbox 아키텍처 완전 동일
