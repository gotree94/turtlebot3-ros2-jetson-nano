# Day 21 — 최종 프로젝트 완성 & 데모

> **목표:** Jetson Nano 4GB 기반 제한된 AI 자율 순찰 시스템을 완성하고
> 최종 데모를 진행한다.

---

## 21.1 최종 체크리스트

### 하드웨어

```
□ Jetson Nano Developer Kit (SD 카드 모듈, 4GB)
□ USB SSD 연결 (또는 고속 SD 카드)
□ TurtleBot3 Burger 조립 + 배터리 충전
□ OpenCR, LDS-01 USB 연결
□ USB 카메라 장착
□ J48 점퍼 설정 (DC 전원 활성화)
□ DC 배럴잭 5V 4A 전원 어댑터 연결
□ 방열판 + 쿨링팬 작동 확인
□ Remote PC (Ubuntu 22.04, ROS2 Humble Desktop)
```

### 소프트웨어

```
□ JetPack 4.6.1 (Ubuntu 18.04 aarch64)
□ Docker Engine + NVIDIA Container Toolkit
□ ROS2 Humble 컨테이너 (dustynv/ros:humble-ros-base-l4t-r32.7.1)
□ TurtleBot3 패키지 빌드 완료 (Docker 내부)
□ Remote PC ROS2 Humble Desktop 설치
□ ROS_DOMAIN_ID=30 설정
□ SLAM 지도 생성 완료 (~/jetson_nano_map)
□ Waypoint YAML 파일 최종 확정
□ patrol_bot_nano 패키지 빌드 완료
□ USB 카메라 (v4l2_camera) 동작 확인
□ CuPy / OpenCV CUDA 설치 (선택)
```

---

## 21.2 최종 데모 시나리오

### 데모 1: OpenCV + CUDA 기본 처리

```
목표: Jetson Nano GPU를 활용한 기본 이미지 처리 시연

절차:
  1. USB 카메라 실행
     ros2 run v4l2_camera v4l2_camera_node
  
  2. Image Processor 실행
     ros2 run patrol_bot_nano image_processor
  
  3. 카메라 앞에 물체 배치
  4. 수동으로 waypoint 도착 트리거:
     ros2 topic pub /waypoint_arrived std_msgs/msg/String "{data: 'test_point'}"
  
  5. 저장된 이미지 확인:
     ls -la ~/patrol_images/
     # patrol_20260531_120000.jpg 등 저장됨

예상 결과:
  - 이미지가 ~/patrol_images/에 저장됨
  - 객체 Contour가 표시된 결과 이미지
  - /patrol_vision_status 토픽에 로그 발행
```

### 데모 2: LIDAR 기반 자율 순찰

```
목표: Nav2 Waypoint 기반 순찰 시스템 시연

절차:
  Remote PC:
    1. Nav2 실행
       ros2 launch turtlebot3_navigation2 navigation2.launch.py \
         map:=~/jetson_nano_map.yaml
  
  Jetson Nano:
    2. Bringup
       export TURTLEBOT3_MODEL=burger
       ros2 launch turtlebot3_bringup robot.launch.py
    
    3. Image Processor
       docker exec -it ros2_humble bash
       ros2 run patrol_bot_nano image_processor
  
  Remote PC:
    4. RViz2에서 Nav Goal 설정
    5. Waypoint Navigator 실행
    6. 순찰 경로 모니터링

예상 결과:
  - Waypoint 도착 시 자동 이미지 캡처
  - 순찰 완료 후 ~/patrol_images/에 이미지 저장됨
  - 순찰 로그 출력
```

### 데모 3: Jetson Nano 성능 모니터링

```
목표: Jetson Nano의 시스템 리소스 사용량 시각화

절차:
  1. jtop 실행
  2. 모든 ROS2 노드 실행 상태에서 GPU/CPU/RAM 사용률 측정

예상 리소스 사용량:
  프로세스              CPU    GPU    RAM
  ─────────────────────────────────────────
  Bringup (LIDAR+OpenCR)   8%     -    200MB
  Docker + ROS2            15%    -    500MB
  Image Processor          20%    15%  200MB
  v4l2_camera              5%     5%   100MB
  OS + 기타                20%     5%   2000MB
  ─────────────────────────────────────────
  합계                    68%    25%   ~3.0GB
  여유                    32%    75%   ~1GB (위험 수준)
```

---

## 21.3 프로젝트 회고

### 성취한 것

```
✅ Jetson Nano 4GB (eMMC 없음) + JetPack 4.6.1 환경 구축
✅ USB SSD 부팅 설정으로 SD 카드 한계 극복
✅ Docker 기반 ROS2 Humble 개발 환경 구축
✅ TurtleBot3 Burger bringup 및 원격 제어
✅ SLAM + Nav2 기반 자율주행 (Remote PC)
✅ OpenCV + CUDA 기본 이미지 처리 (제한적 GPU 활용)
✅ Waypoint 순찰 + 이미지 캡처 통합
✅ 개발용/양산용 모듈 차이 이해
```

### Jetson Nano의 한계와 극복

| 한계 | 극복 방법 | 대안 플랫폼 |
|------|----------|-----------|
| 4GB RAM 부족 | Docker + 네이티브 분리, 경량 프로세스 | Jetson Orin Nano (8GB+) |
| eMMC 없음 | USB SSD 부팅으로 성능 확보 | eMMC 모듈 구매 |
| Maxwell 128-core GPU | 경량 모델, 기본 OpenCV | Jetson Orin Nano (1024-core) |
| CUDA 10.2 (구버전) | 호환 라이브러리 사용 | Jetson Orin (CUDA 12.x) |
| Ubuntu 18.04 | Docker로 ROS2 Humble 격리 | Jetson Orin (Ubuntu 22.04) |

### Jetson Nano가 적합한 경우

```
✅ AI/ML 학습 입문자 (CUDA 첫 경험)
✅ 예산이 제한된 교육 환경 (~$99)
✅ 기존 JetPack 4.x 프로젝트 유지보수
✅ 간단한 GPU 가속 임베디드 시스템
✅ ROS2 + 제한적 AI 결합 학습
```

---

## 21.4 다음 단계

### Jetson Nano에서 더 도전하기

1. **TensorRT 최적화**: ONNX 모델 → TensorRT 엔진 변환 (INT8 양자화)
2. **YOLOv2 Tiny**: 128x128 입력으로 경량 객체 탐지 (~2-3 FPS)
3. **멀티 카메라**: 2개 CSI 카메라로 스테레오 비전
4. **원격 모니터링**: 저장된 이미지를 웹 대시보드로 전송
5. **멀티 로봇**: 2대의 TurtleBot3 + Jetson Nano 협업

### Jetson Orin Nano로 업그레이드

```
Jetson Nano (~$99)          →      Jetson Orin Nano (~$499)
──────────────────────────────────────────────────────────────
128-core Maxwell GPU        →      1024-core Ampere GPU
472 GFLOPS                  →      40 TOPS (85배 향상)
4GB RAM                     →      8GB/16GB LPDDR5
Ubuntu 18.04 + Docker ROS2  →      Ubuntu 22.04 + 네이티브 ROS2
CUDA 10.2                   →      CUDA 12.x
YOLOv2 Tiny (~2FPS)         →      YOLOv8 (~60FPS)
```

---

## 21.5 최종 소감

> **🎉 Congratulations! You've completed the TurtleBot3 ROS2 Humble curriculum on Jetson Nano 4GB!**
>
> 이 과정에서 당신은:
> - Jetson Nano의 **SD 카드 모듈(개발용)과 eMMC 모듈(양산용)의 차이**를 이해했습니다
> - **USB SSD 부팅**으로 SD 카드의 성능 한계를 극복했습니다
> - **Docker 기반 ROS2 Humble** 환경을 구축했습니다
> - 제한된 **Maxwell 128-core GPU**로 기초 AI/컴퓨터 비전을 경험했습니다
> - TurtleBot3 Burger로 **SLAM, Nav2 자율주행, Waypoint 순찰**을 구현했습니다
>
> Jetson Nano는 AI 로봇 공학 입문을 위한 **가장 경제적인 CUDA 플랫폼**입니다.
> 이 기초를 바탕으로 더 강력한 Jetson Orin 시리즈나 다른 고급 플랫폼으로
> 나아가길 바랍니다!
