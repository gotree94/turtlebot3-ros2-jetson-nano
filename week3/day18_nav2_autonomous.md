# Day 18 — Nav2 자율주행 실습

> **RPi 버전 Day 18과 내용이 동일합니다.**

---

## 실행 구성

```
Remote PC (Ubuntu 22.04)
┌────────────────────────────────────────────────┐
│  1. navigation2.launch.py (Nav2 전체 실행)      │
│  2. RViz2 (2D Pose Estimate + 2D Nav Goal)     │
│  3. AMCL 로봇 위치 추정                         │
│  4. 경로 계획 및 추종                           │
└──────────────────┬─────────────────────────────┘
                   │ ROS_DOMAIN_ID=30
┌──────────────────▼─────────────────────────────┐
│  Jetson Nano (네이티브)                         │
│  - Bringup 실행 (LIDAR + OpenCR)               │
│  - /cmd_vel 수신 → 모터 제어                   │
│  - /scan, /odom → Remote PC                   │
└────────────────────────────────────────────────┘
```

> **전체 내용:** `../../turtlebot3-ros2-curriculum/week3/day18_nav2_autonomous.md` 참조
>
> 실습: Nav2 자율주행 실행, 2D Pose Estimate, 2D Nav Goal, Waypoint 완전 동일
