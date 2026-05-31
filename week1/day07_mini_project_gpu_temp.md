# Day 7 — 미니 프로젝트: GPU/CPU 온도 모니터링

> **목표:** Jetson Nano의 GPU와 CPU 온도를 ROS2 토픽으로 발행하는 노드를 작성한다.

---

## 7.1 프로젝트 개요

Jetson Nano의 온도 센서는 CPU, GPU, 보드, 다이오드 등 여러 지점을 측정합니다.
이 데이터를 ROS2 토픽(`/jetson_temps`)으로 발행하고 모니터링하는 시스템을 구축합니다.

```
┌─────────────────┐    /jetson_temps    ┌──────────────────────┐
│  temp_monitor   │ ──────────────────►  │  Remote PC          │
│  (Docker)       │   Float64MultiArray  │  ros2 topic echo    │
│                 │                      │  또는 rqt_plot       │
│  - GPU 온도      │                      │                      │
│  - CPU 온도      │                      │  ros2 bag record    │
│  - 보드 온도      │                      │  (데이터 저장)       │
│  - 다이오드 온도   │                      │                      │
└─────────────────┘                      └──────────────────────┘
```

---

## 7.2 네이티브에서 온도 센서 읽기

Docker 컨테이너는 호스트의 `/sys` 디렉토리에 접근해야 하므로,
`--privileged` 또는 볼륨 마운트가 필요합니다.

```bash
# 네이티브에서 온도 확인
cat /sys/devices/virtual/thermal/thermal_zone*/temp
# 출력 (단위: 밀리섭씨 / 1000 = °C)
# 55000  → 55°C (CPU)
# 53000  → 53°C (GPU)
# 41000  → 41°C (보드)
# 61500  → 61.5°C (다이오드)

# 센서 타입 확인
cat /sys/devices/virtual/thermal/thermal_zone*/type
# CPU-therm
# GPU-therm
# Tboard_therm
# Tdiode_therm
```

---

## 7.3 ROS2 온도 모니터링 노드

`~/ros2_ws/src/temp_monitor/temp_monitor/temp_monitor.py`:

```python
#!/usr/bin/env python3
"""
Jetson Nano GPU/CPU 온도 모니터링 ROS2 노드
- /sys/class/thermal에서 온도 읽기
- /jetson_temps 토픽으로 발행
- 1초 간격 갱신
"""

import rclpy
from rclpy.node import Node
from std_msgs.msg import Float64MultiArray, MultiArrayDimension
import glob


class TempMonitor(Node):
    """Jetson Nano 온도 모니터링"""

    def __init__(self):
        super().__init__('temp_monitor')

        # Publisher
        self.pub = self.create_publisher(
            Float64MultiArray, '/jetson_temps', 10)

        # Timer: 1초 간격
        self.create_timer(1.0, self.publish_temps)

        # Thermal zone 파일 찾기
        self.thermal_zones = sorted(
            glob.glob('/sys/devices/virtual/thermal/thermal_zone*'))
        self.sensor_names = []

        for tz in self.thermal_zones:
            try:
                with open(f'{tz}/type', 'r') as f:
                    self.sensor_names.append(f.read().strip())
            except:
                self.sensor_names.append('unknown')

        self.get_logger().info(
            f'🌡️ TempMonitor started: {self.sensor_names}')

    def read_temps(self):
        """모든 온도 센서 읽기 (밀리섭씨 → 섭씨)"""
        temps = []
        for tz in self.thermal_zones:
            try:
                with open(f'{tz}/temp', 'r') as f:
                    temp_c = float(f.read().strip()) / 1000.0
                    temps.append(temp_c)
            except:
                temps.append(0.0)
        return temps

    def publish_temps(self):
        """온도 데이터 발행"""
        temps = self.read_temps()
        if not temps:
            return

        # Float64MultiArray 메시지 생성
        msg = Float64MultiArray()
        msg.layout.dim.append(MultiArrayDimension())
        msg.layout.dim[0].label = 'temperature'
        msg.layout.dim[0].size = len(temps)
        msg.layout.dim[0].stride = len(temps)
        msg.data = temps

        self.pub.publish(msg)

        # 로그 출력
        temp_str = ' | '.join(
            [f'{n}: {t:.1f}°C' for n, t in zip(self.sensor_names, temps)])
        self.get_logger().info(f'🌡️ {temp_str}')


def main(args=None):
    rclpy.init(args=args)
    node = TempMonitor()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.shutdown()


if __name__ == '__main__':
    main()
```

**setup.py에 추가:**
```python
entry_points={
    'console_scripts': [
        'temp_monitor = temp_monitor.temp_monitor:main',
    ],
},
```

---

## 7.4 실행 방법

```bash
# 1. 컨테이너에서 호스트 /sys 접근 권한 확인
# Docker 실행 시 --privileged 옵션이 없으면 추가
docker stop ros2_humble
docker rm ros2_humble

docker run -itd \
  --name ros2_humble \
  --gpus all \
  --runtime nvidia \
  -e DISPLAY=$DISPLAY \
  -e ROS_DOMAIN_ID=30 \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -v ~/ros2_ws:/ros2_ws \
  -v /dev:/dev \
  -v /sys:/sys \
  --network host \
  --privileged \
  dustynv/ros:humble-ros-base-l4t-r32.7.1

# 2. 컨테이너 접속 및 빌드
docker exec -it ros2_humble bash
source /opt/ros/humble/setup.bash
cd /ros2_ws
colcon build --packages-select temp_monitor
source install/setup.bash

# 3. 실행
ros2 run temp_monitor temp_monitor

# 출력 예시:
# [INFO] [temp_monitor]: 🌡️ CPU-therm: 55.0°C | GPU-therm: 52.0°C | ...
```

---

## 7.5 Remote PC에서 모니터링

```bash
# Remote PC에서 온도 확인
ros2 topic echo /jetson_temps

# 데이터 기록
ros2 bag record /jetson_temps -o temp_data

# rqt_plot으로 실시간 그래프 (Remote PC)
rqt_plot /jetson_temps/data[0] /jetson_temps/data[1]
```

---

## 7.6 GPU 부하 테스트

```bash
# CUDA 샘플로 GPU 부하 생성
/usr/local/cuda-10.2/samples/bin/aarch64/linux/release/nbody -benchmark -numbodies=256000

# 별도 터미널에서 온도 변화 관찰
docker exec -it ros2_humble bash
ros2 run temp_monitor temp_monitor
# GPU 온도가 50°C → 70°C+까지 상승하는 것 확인
```

---

## 7.7 프로젝트 완료 체크리스트

- [ ] 온도 모니터링 노드 정상 실행 (1초 간격 발행)
- [ ] `/jetson_temps` 토픽 확인 (`ros2 topic echo`)
- [ ] 4개 이상 온도 센서 값 확인 (CPU, GPU, 보드, 다이오드)
- [ ] GPU 부하 전/후 온도 차이 기록
- [ ] `ros2 bag`으로 온도 데이터 저장
- [ ] Remote PC에서 rqt_plot으로 그래프 확인

---

## 📝 연습 문제

1. GPU 부하 전후의 온도 차이를 측정하고 기록하세요 (최소 5분 간격)
2. 5W 모드와 10W 모드에서 각각 부하 테스트를 수행하고 온도 차이를 비교하세요
3. 온도 데이터를 CSV 파일로 저장하는 기능을 추가하세요
4. 특정 온도(예: 75°C) 이상일 때 경고 메시지를 발행하는 기능을 추가하세요
5. `tegrastats`의 온도 데이터와 비교하여 정확성을 검증하세요
