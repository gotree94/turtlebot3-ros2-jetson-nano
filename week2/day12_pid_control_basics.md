# Day 12 — PID 속도 제어 기초

> **RPi 버전 Day 12와 내용이 동일합니다.**
>
> Jetson Nano에서 PID 제어는 Docker 컨테이너 내부에서 개발/실행합니다.
> `cmd_vel` 토픽은 Docker 컨테이너 → 네이티브 → TurtleBot3로 전달됩니다.

---

## 네트워크 토폴로지

```
┌─────────────────┐  /cmd_vel (ROS2)    ┌──────────────────┐
│  Docker         │ ──────────────────► │  Jetson Nano     │
│  (ROS2 Humble)  │                     │  (네이티브)       │
│                 │                     │                  │
│  PID Controller │                     │  Bringup         │
│  Go-to-Goal     │                     │  → OpenCR        │
│  cmd_vel pub    │                     │  → 모터 제어      │
└─────────────────┘                     └──────────────────┘
```

> **전체 내용:** `../../turtlebot3-ros2-curriculum/week2/day12_pid_control_basics.md` 참조
>
> 실습: PID 이론, Go-to-Goal 노드 작성, 게인 튜닝 완전 동일
