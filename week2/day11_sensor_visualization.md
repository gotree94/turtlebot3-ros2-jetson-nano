# Day 11 — 센서 데이터 시각화 & RViz2

> **RPi 버전 Day 11과 내용이 동일합니다.**
>
> RViz2는 **Remote PC**에서 실행합니다. Jetson Nano의 Maxwell GPU는
> 3D PointCloud/LaserScan 렌더링에 충분한 성능을 내기 어렵습니다.

---

## 실행 구성

```
┌─────────────────┐  /scan, /odom, /tf   ┌──────────────────┐
│  Jetson Nano    │ ◄───────────────────► │  Remote PC       │
│  (네이티브)      │                       │                  │
│                 │                       │  RViz2           │
│  Bringup 실행   │                       │  rqt             │
│  센서 데이터 발행│                       │  ros2 topic echo  │
└─────────────────┘                       └──────────────────┘
```

> **전체 내용:** `../../turtlebot3-ros2-curriculum/week2/day11_sensor_visualization.md` 참조
>
> 실습: /scan, /odom, /tf 토픽 확인, RViz2 설정, LaserScan 시각화 완전 동일
