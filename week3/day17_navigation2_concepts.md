# Day 17 — Navigation2 개념

> **RPi 버전 Day 17과 내용이 동일합니다.**

---

## Nav2 실행 위치

SLAM과 마찬가지로 Nav2 시각화는 **Remote PC에서 실행**합니다.

```
┌─────────────────┐  /scan, /odom, /tf ┌──────────────────────┐
│  Jetson Nano    │ ◄──────────────── │  Remote PC           │
│  (네이티브)      │                    │                      │
│                 │  /cmd_vel          │  Nav2 Stack          │
│  Bringup 실행   │ ────────────────► │  (Planner+AMCL+      │
│                 │                    │   Controller)        │
│                 │                    │  RViz2 시각화        │
└─────────────────┘                    └──────────────────────┘
```

> **전체 내용:** `../../turtlebot3-ros2-curriculum/week3/day17_navigation2_concepts.md` 참조
>
> 이론 내용: Nav2 아키텍처, Planner/Controller/AMCL/BT 개념 완전 동일
