# Week 2 연습 문제 (Jetson Nano 4GB)

> **기본 문제:** `../../turtlebot3-ros2-curriculum/exercises/week2_exercises.md` 참조
>
> 아래는 Jetson Nano 4GB에 특화된 추가 문제입니다.

---

## Jetson Nano 추가 문제

### 문제 1: Docker vs 네이티브 Bringup 성능 비교

TurtleBot3 Bringup을 Docker 컨테이너 내부와 네이티브에서 각각 실행하고
LIDAR 스캔 데이터의 지연 시간을 비교하세요.

```bash
# Docker Bringup
docker exec -it ros2_humble bash
ros2 launch turtlebot3_bringup robot.launch.py

# 네이티브 Bringup
# (다른 SSH 세션)
ros2 launch turtlebot3_bringup robot.launch.py

# 각각의 /scan 토픽 지연 시간 측정
ros2 topic hz /scan
```

### 문제 2: 제한된 RAM 환경에서 colcon build 최적화

```bash
# 4GB RAM에서 빌드 시 메모리 부족 방지
colcon build --symlink-install \
  --parallel-workers 2 \        # 병렬 작업 수 제한
  --cmake-args -DCMAKE_BUILD_TYPE=Release

# 빌드 중 메모리 사용량 모니터링
htop
```

### 문제 3: SD 카드 vs USB SSD 벤치마크

SD 카드와 USB SSD 각각에서 다음 작업의 수행 시간을 측정하고 비교하세요:

1. `colcon build --packages-select turtlebot3_msgs` 빌드 시간
2. `ros2 bag record /scan` 60초 녹화 후 파일 크기
3. `find / -name "*.py"` 검색 시간

### 문제 4: Remote PC 의존성 측정

Remote PC와 Jetson Nano 간 네트워크 지연이 TurtleBot3 제어에 미치는 영향을 측정:

```bash
# Remote PC에서 cmd_vel 발행 → Jetson Nano 수신까지 지연
# Jetson Nano:
ros2 topic echo /cmd_vel --once --csv

# Remote PC:
ros2 topic pub /cmd_vel geometry_msgs/Twist \
  '{linear: {x: 0.1}}' -1
```

지연 시간이 100ms를 초과하는 경우 개선 방법을 제안하세요.
