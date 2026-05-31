# Day 3 — Remote PC 세팅

> **RPi 버전 Day 3과 내용이 동일합니다.**
>
> Jetson Nano는 온보드 GPU로 RViz2를 원활히 실행하기 어려우므로
> **Remote PC가 필수입니다.** RPi 버전과 동일한 Remote PC 설정을 따릅니다.

---

## Remote PC 환경

| 항목 | 사양 |
|------|------|
| OS | Ubuntu 22.04 LTS Desktop (amd64) |
| ROS2 | Humble Desktop (네이티브 apt 설치) |
| 목적 | RViz2, rqt, SLAM/Nav2 시각화 |
| 연결 | WiFi (동일 ROS_DOMAIN_ID=30) |

> **전체 내용:** `../turtlebot3-ros2-curriculum/week1/day03_remote_pc_setup.md` 참조
>
> 실습: Ubuntu 22.04 설치, ROS2 Humble Desktop 설치 (apt), ROS_DOMAIN_ID 설정
