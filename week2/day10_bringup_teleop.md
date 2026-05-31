# Day 10 — TurtleBot3 Bringup & Teleoperation

> **RPi 버전 Day 10과 내용이 동일합니다.**
>
> Jetson Nano에서는 Bringup을 **네이티브**에서 실행하고 (USB/시리얼 안정성),
> Teleop은 **Remote PC**에서 실행하는 것이 권장됩니다.

---

## 실행 구성

```
┌─────────────────┐    ROS_DOMAIN_ID=30    ┌──────────────────┐
│  Jetson Nano    │ ◄────────────────────► │  Remote PC       │
│  (네이티브)      │                        │  (Ubuntu 22.04)  │
│                 │                        │                   │
│  Bringup        │   /scan, /odom,        │  teleop_keyboard  │
│  LIDAR 드라이버 │   /cmd_vel, /tf        │  RViz2            │
│  OpenCR 제어    │                        │                   │
└─────────────────┘                        └──────────────────┘
```

> **전체 내용:** `../../turtlebot3-ros2-curriculum/week2/day10_bringup_teleop.md` 참조
>
> 실습: Bringup, Teleop, Topic 확인 완전 동일 (실행 위치만 다름)
