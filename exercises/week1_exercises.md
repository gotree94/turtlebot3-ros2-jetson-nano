# Week 1 연습 문제 (Jetson Nano 4GB)

> **기본 문제:** `../../turtlebot3-ros2-curriculum/exercises/week1_exercises.md` 참조
>
> 아래는 Jetson Nano 4GB에 특화된 추가 문제입니다.

---

## Jetson Nano 추가 문제

### 문제 1: SD 카드 모듈 vs eMMC 모듈 식별

자신의 Jetson Nano가 어떤 모듈인지 확인하는 명령어를 작성하고 결과를 설명하세요.

```bash
# 힌트
lsblk
# eMMC가 있으면 mmcblk0 등이 표시됨
# SD 카드 모듈이면 mmcblk1만 표시
```

### 문제 2: USB SSD 부팅 확인

```bash
# 현재 root 파일시스템이 어느 장치에 있는지 확인
df -h /

# SD 카드: /dev/mmcblk0p1
# USB SSD: /dev/sda1
```

USB SSD 부팅 시 속도 이점을 측정하는 스크립트를 작성하세요.

### 문제 3: Docker 메모리 사용량

```bash
# Docker 컨테이너 메모리 사용량 확인
docker stats ros2_humble --no-stream

# ROS2 Humble talker/listener 실행 전후 메모리 차이 측정
```

### 문제 4: 전원 모드별 성능 비교

Jetson Nano의 5W 모드와 10W 모드에서 각각 다음 작업을 수행하고 차이를 기록하세요:

1. CPU 벤치마크 (sysbench)
2. GPU 부하 (CUDA 샘플 nbody)
3. Docker 컨테이너 응답 시간
4. 전력 소비 차이 (jtop 전력 탭)

### 문제 5: CUDA 10.2 Compute Capability

Jetson Nano의 Maxwell GPU Compute Capability (5.3)이 지원하는 기능을 조사하세요:

- FP16 지원 여부
- INT8 Tensor Core 유무
- Dynamic Parallelism 지원
- Unified Memory 지원
