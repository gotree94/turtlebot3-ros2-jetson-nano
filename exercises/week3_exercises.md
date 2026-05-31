# Week 3 연습 문제 (Jetson Nano 4GB)

> **기본 문제:** `../../turtlebot3-ros2-curriculum/exercises/week3_exercises.md` 참조
>
> 아래는 Jetson Nano 4GB에 특화된 추가 문제입니다.

---

## Jetson Nano 추가 문제

### 문제 1: SLAM 메모리 최적화

Jetson Nano의 4GB RAM에서 slam_toolbox를 실행하기 위한 파라미터 최적화:

```yaml
# 최적화된 slam_toolbox 파라미터 (메모리 사용량 감소)
slam_params:
  minimum_travel_distance: 0.3     # 기본 0.5
  minimum_travel_heading: 0.3      # 기본 0.5
  resolution: 0.05                  # 지도 해상도 (0.05=5cm)
  map_update_interval: 2.0         # 지도 업데이트 간격 (초)
  max_laser_range: 3.5             # LIDAR 최대 범위 제한 (3.5m)
```

**과제:** 각 파라미터가 메모리 사용량에 미치는 영향을 측정하고 설명하세요.

### 문제 2: GPU 메모리 한계 극복

Jetson Nano의 공유 메모리(4GB CPU+GPU 공유) 환경에서
GPU 메모리 부족을 진단하고 해결하는 방법:

```bash
# GPU 메모리 사용량 확인
cat /sys/devices/7000c000.ahub/7010c000.i2c/i2c-6/6-0040/i2c-8/8-0041/power_monitor/voltage*

# 또는
jtop  # MEM 탭에서 GPU 메모리 확인

# GPU 메모리 부족 시 해결:
# 1. 불필요한 GPU 프로세스 종료
# 2. OpenCV Mat을 CPU 메모리로 유지
# 3. 이미지 해상도 다운샘플링 (640x480 → 320x240)
```

### 문제 3: TensorRT 경량 모델 변환

Jetson Nano에서 TensorRT로 경량 모델을 최적화하는 과정을 설계하세요:

```python
# 의사 코드
# 1. ONNX 모델 준비 (예: SqueezeNet, MobileNet)
# 2. TensorRT로 INT8 양자화
# 3. 추론 속도 측정 (CPU vs GPU vs TensorRT)

# 참고: jetson-inference 라이브러리 활용
# git clone https://github.com/dusty-nv/jetson-inference
```

### 문제 4: Jetson Nano vs RPi 프로젝트 비교

Jetson Nano 4GB로 구현한 최종 프로젝트와
RPi 4B 8GB로 구현한 프로젝트의 차이점을 비교하는 보고서를 작성하세요:

| 항목 | RPi 4B | Jetson Nano 4GB |
|------|--------|-----------------|
| SLAM 매핑 | Remote PC | Remote PC (동일) |
| Nav2 자율주행 | Remote PC | Remote PC (동일) |
| GPU 가속 | 없음 | 제한적 (128-core) |
| 이미지 처리 | CPU only | OpenCV CUDA |
| 객체 탐지 | 불가 | YOLOv2 Tiny (제한적) |
| Docker | 불필요 | 필수 (ROS2 Humble) |
| 스토리지 병목 | SD 카드 | USB SSD (개선 가능) |
