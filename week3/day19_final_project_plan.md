# Day 19 — 최종 프로젝트 기획: 제한된 AI 자율 순찰

> **목표:** Jetson Nano 4GB의 제한된 GPU(Maxwell 128-core)를 고려하여
> 실현 가능한 AI 기반 자율 순찰 로봇을 설계한다.

---

## 19.1 Jetson Nano AI 한계 인식

| 항목 | Jetson Nano | Jetson Orin Nano |
|------|------------|-------------------|
| GPU | Maxwell 128-core @921MHz | Ampere 1024-core @1GHz |
| AI 성능 | 472 GFLOPS (FP16) | 40 TOPS (INT8) |
| GPU 메모리 | 공유 4GB (CPU와 공유) | 전용 8GB/16GB |
| YOLO 모델 | YOLOv2 Tiny (~2FPS) | YOLOv8n (~60FPS) |
| TensorRT | 지원 (제한적) | 완벽 지원 |
| 실시간 추론 | 간단한 분류만 가능 | 복합 객체 탐지 가능 |

> ⚠️ **Jetson Nano로는 실시간 객체 탐지가 매우 제한적입니다.**
> 대신 다음 전략을 사용합니다:
> - **경량 모델** 사용 (YOLOv2 Tiny, MobileNet, SqueezeNet)
> - **정지 상태에서만 추론** (이동 중에는 탐지 불가)
> - **단순한 시각적 판단** (객체 유무, 색상 인식 등)
> - **OpenCV + GPU 기본 처리** 위주

---

## 19.2 프로젝트 범위 조정

### 가능한 기능 (✅)
- ✅ LIDAR 기반 자율 순찰 (Nav2 Waypoint)
- ✅ 정지 상태에서 간단한 이미지 캡처 및 저장
- ✅ OpenCV 기본 이미지 처리 (cvtColor, threshold, contour)
- ✅ GPU 가속 (CUDA) 기본 연산 (벡터 연산, 필터)
- ✅ 사전 학습된 경량 CNN 모델로 객체 분류
- ✅ 순찰 로그 기록 (이미지 + LIDAR 데이터)

### 불가능한 기능 (❌)
- ❌ 실시간 객체 탐지 (YOLOv8, SSD, Faster R-CNN)
- ❌ 연속적인 비디오 스트림 AI 분석
- ❌ 복잡한 딥러닝 모델 학습 (4GB RAM 한계)
- ❌ RViz2 + AI 동시 실행

---

## 19.3 시스템 아키텍처

```
┌──────────────────────────────────────────────────┐
│  Jetson Nano                                     │
│                                                  │
│  Docker (ROS2 Humble)                            │
│  ┌────────────────┐  ┌──────────────────────┐    │
│  │ Waypoint       │  │ Camera + OpenCV Node │    │
│  │ Navigator      │  │ (정지 시 캡처)         │    │
│  └───────┬────────┘  └──────────┬───────────┘    │
│          │                      │                │
│          └──────┬───────────────┘                │
│                 │                                │
│  ┌──────────────▼────────────────────────────┐   │
│  │         Patrol Monitor                    │   │
│  │  (로그 + 이미지 저장 + 순찰 상태)          │   │
│  └──────────────────────────────────────────┘   │
│                                                  │
│  네이티브                                        │
│  ┌──────────────────────────────────────────┐   │
│  │  Bringup (LIDAR + OpenCR)                 │   │
│  └──────────────────────────────────────────┘   │
└──────────────────────┬──────────────────────────┘
                       │ ROS_DOMAIN_ID=30
┌──────────────────────▼──────────────────────────┐
│  Remote PC                                      │
│  - RViz2 (순찰 경로 시각화)                      │
│  - 저장된 이미지 확인                             │
│  - 순찰 로그 모니터링                            │
└──────────────────────────────────────────────────┘
```

---

## 19.4 개발 일정

### Day 19: 기획 (오늘)
- [ ] 시스템 아키텍처 설계 (위 다이어그램)
- [ ] 카메라 노드 테스트 (USB 카메라)
- [ ] OpenCV + CUDA 기본 연산 테스트
- [ ] Waypoint YAML 파일 작성

### Day 20: 구현
- [ ] OpenCV + ROS2 이미지 처리 노드 (정지 시 캡처)
- [ ] CUDA 기본 연산 테스트
- [ ] Waypoint Navigator 통합
- [ ] 순찰 로그 노드 구현

### Day 21: 완성
- [ ] 전체 통합 및 데모
- [ ] 순찰 → 정지 → 캡처 → 로깅 자동화
- [ ] 최종 시연

---

## 19.5 준비

```bash
# USB 카메라 연결 확인
ls /dev/video*

# OpenCV + CUDA 확인 (컨테이너)
docker exec -it ros2_humble bash
python3 -c "import cv2; print(cv2.getBuildInformation())" | grep CUDA
# CUDA 여부 확인 (JetPack OpenCV는 CUDA 지원)

# CUDA 기본 연산 테스트
python3 -c "
import numpy as np
import time

# CPU
a = np.random.rand(1000, 1000).astype(np.float32)
b = np.random.rand(1000, 1000).astype(np.float32)
t0 = time.time()
for _ in range(100):
    c = a @ b
print(f'CPU: {(time.time()-t0)/100*1000:.2f}ms')

# GPU (CuPy 설치 필요)
try:
    import cupy as cp
    a_gpu = cp.asarray(a)
    b_gpu = cp.asarray(b)
    t0 = time.time()
    for _ in range(100):
        c_gpu = a_gpu @ b_gpu
    cp.cuda.Stream.null.synchronize()
    print(f'GPU: {(time.time()-t0)/100*1000:.2f}ms')
except ImportError:
    print('CuPy not installed')
"
```

---

## 📝 기획 문제

1. Jetson Nano의 GPU 한계를 고려할 때, 실현 가능한 AI 기능 3가지를 나열하세요
2. 실시간 객체 탐지가 불가능한 환경에서 대안으로 사용할 수 있는 방법을 2가지 이상 제시하세요
3. 순찰 중 정지 상태에서만 이미지를 캡처하는 이유를 설명하세요
4. 프로젝트의 성공 기준을 정의하세요 (무엇을, 어떻게 측정할 것인가)
5. Jetson Nano 4GB에서 Docker, Bringup, OpenCV를 동시에 실행할 때의 메모리 사용량을 예측하세요
