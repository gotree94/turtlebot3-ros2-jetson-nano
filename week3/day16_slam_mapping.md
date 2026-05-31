# Day 16 — 실제 SLAM 매핑

> **RPi 버전 Day 16과 내용이 동일합니다.**

Jetson Nano의 4GB RAM으로는 slam_toolbox + RViz2 동시 온보드 실행이 어렵습니다.
**Remote PC에서 slam_toolbox와 RViz2를 실행**합니다.

---

## 매핑 실행 구성

```
┌─────────────────┐  /scan             ┌──────────────────────┐
│  Jetson Nano    │ ◄──────────────── │  Remote PC           │
│  (네이티브)      │    /odom           │                      │
│                 │    /tf             │  1. slam_toolbox 실행 │
│  Bringup 실행   │ ────────────────► │     (매핑 수행)       │
│  LIDAR + OpenCR │                   │  2. RViz2 시각화     │
│                 │                   │  3. 키보드 텔레옵     │
│                 │                   │  4. 지도 저장         │
└─────────────────┘                   └──────────────────────┘
```

> **전체 내용:** `../../turtlebot3-ros2-curriculum/week3/day16_slam_mapping.md` 참조
>
> 실습: slam_toolbox 실행, RViz2 매핑, 지도 저장 완전 동일
