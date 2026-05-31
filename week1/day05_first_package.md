# Day 5 — 첫 ROS2 패키지

> **RPi 버전 Day 5와 내용이 동일합니다.**
>
> Jetson Nano에서는 Docker 컨테이너 내부에서 패키지를 생성하고 빌드합니다.
> 볼륨 마운트된 `~/ros2_ws` 디렉토리를 사용하므로,
> 소스 코드는 네이티브에서도 편집 가능합니다.

---

## Docker 환경에서 패키지 빌드

```bash
# 1. 컨테이너 접속
docker exec -it ros2_humble bash
source /opt/ros/humble/setup.bash

# 2. 워크스페이스 확인
cd /ros2_ws
ls -la
# src/ 디렉토리 확인

# 3. 패키지 생성
ros2 pkg create my_first_pkg \
  --build-type ament_python \
  --dependencies rclpy

# 4. 노드 작성 (src/my_first_pkg/my_first_pkg/my_publisher.py)
# RPi 버전과 동일한 코드 사용

# 5. 빌드
cd /ros2_ws
colcon build --packages-select my_first_pkg

# 6. 실행
source install/setup.bash
ros2 run my_first_pkg my_publisher
```

> ⚠️ **colcon build 속도:**
> - USB SSD: ~5~10초
> - SD 카드: ~30~60초 (볼륨 마운트 I/O 병목)
> - **USB SSD 사용을 강력 권장합니다**

> **전체 내용:** `../turtlebot3-ros2-curriculum/week1/day05_first_package.md` 참조
>
> 실습: Python publisher/subscriber 작성, setup.py 설정, colcon build 완전 동일
