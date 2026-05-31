# Day 4 — ROS2 핵심 개념

> **RPi 버전 Day 4와 내용이 동일합니다.**
>
> 모든 실습은 Docker 컨테이너 내부에서 실행합니다.
> 멀티 터미널이 필요한 경우 `docker exec -it ros2_humble bash`로
> 여러 세션을 열어 진행하세요.

---

## Docker 환경에서 ROS2 실습

```bash
# 터미널 1: 컨테이너 접속
docker exec -it ros2_humble bash
source /opt/ros/humble/setup.bash

# 터미널 2: 다른 세션
docker exec -it ros2_humble bash
source /opt/ros/humble/setup.bash

# rqt 등 GUI는 Remote PC에서 실행
# Remote PC:
ros2 run rqt_graph rqt_graph
```

> **전체 내용:** `../turtlebot3-ros2-curriculum/week1/day04_ros2_concepts.md` 참조
>
> 실습: Topic, Service, Action 통신, pub/sub 패키지 실행 완전 동일
