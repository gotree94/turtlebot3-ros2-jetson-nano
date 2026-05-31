# Week 2 — TurtleBot3 구동 & 센서/제어 (Jetson Nano 4GB)

> **목표:** TurtleBot3 Burger를 실제로 구동하고, LIDAR/IMU/Odometry 센서를 시각화하며 PID 제어까지 마스터한다.

---

## 📅 주간 일정

| 일차 | 주제 | 실행 위치 |
|------|------|----------|
| Day 8 | TurtleBot3 SBC 패키지 설치 | Jetson Nano (Docker + 네이티브 혼합) |
| Day 9 | Remote PC 패키지 설치 | Remote PC |
| Day 10 | Bringup & Teleoperation | Jetson Nano (네이티브 권장) |
| Day 11 | 센서 시각화 (RViz2) | **Remote PC** |
| Day 12 | PID 속도 제어 | Jetson Nano (Docker) |
| Day 13 | URDF & RViz2 심화 | **Remote PC** |
| Day 14 | 미니 프로젝트: 장애물 회피 | Jetson Nano (Docker) + Remote PC 시각화 |

---

## ⚠️ Week 2 핵심 주의사항

| 항목 | 내용 |
|------|------|
| **Bringup** | 네이티브에서 직접 실행 (Docker보다 USB/시리얼 접근이 안정적) |
| **RViz2** | **Remote PC**에서 실행 (Jetson Nano GPU로는 RViz2 3D 렌더링 무리) |
| **colcon build** | Docker 컨테이너 내부에서 실행 (볼륨 마운트된 ~/ros2_ws 사용) |
| **USB SSD 권장** | SD 카드에서 colcon build 시 5~10배 느림 |
| **ROS_DOMAIN_ID** | Jetson Nano와 Remote PC 모두 `ROS_DOMAIN_ID=30` 필수 |
