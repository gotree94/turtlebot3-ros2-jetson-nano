# Day 20 — 최종 프로젝트 구현: 경량 AI + 순찰 통합

> **목표:** Jetson Nano의 제한된 GPU를 활용한 경량 이미지 처리와
> Waypoint 순찰 시스템을 통합한다.

---

## 20.1 OpenCV + CUDA 기본 처리 노드

Jetson Nano의 GPU를 활용한 간단한 이미지 처리 노드:

`~/ros2_ws/src/patrol_bot_nano/patrol_bot_nano/image_processor.py`:

```python
#!/usr/bin/env python3
"""
Jetson Nano 경량 이미지 처리 노드

제한된 GPU(Maxwell 128-core)를 활용한 기본 이미지 처리:
1. USB 카메라 이미지 캡처
2. OpenCV CUDA 기본 연산 (선택적)
3. Waypoint 도착 시 이미지 저장
4. 객체 유무 간단 판단 (Contour 분석)
"""

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from std_msgs.msg import String, Bool
from cv_bridge import CvBridge
import cv2
import numpy as np
import os
from datetime import datetime


class PatrolImageProcessor(Node):
    """
    Jetson Nano Patrol 이미지 프로세서
    
    - Waypoint 도착 시 이미지 캡처 및 저장
    - OpenCV 기본 분석 (객체 유무)
    - GPU 가속 (선택적 CuPy)
    """

    def __init__(self):
        super().__init__('patrol_image_processor')

        # Publisher
        self.status_pub = self.create_publisher(
            String, '/patrol_vision_status', 10)
        self.motion_pub = self.create_publisher(
            Bool, '/object_detected', 10)

        # Subscriber: 카메라 이미지
        self.sub = self.create_subscription(
            Image, '/image_raw', self.image_callback, 10)
        self.bridge = CvBridge()

        # Subscriber: Waypoint 도착 신호
        self.waypoint_sub = self.create_subscription(
            String, '/waypoint_arrived', self.waypoint_callback, 10)

        # 저장 디렉토리
        self.save_dir = os.path.expanduser('~/patrol_images')
        os.makedirs(self.save_dir, exist_ok=True)

        # 상태
        self.last_image = None
        self.capture_on_next = False

        # GPU 확인
        self.use_gpu = self.check_gpu()
        if self.use_gpu:
            self.get_logger().info('✅ GPU acceleration available')
        else:
            self.get_logger().info('⚠️ GPU acceleration not available, using CPU')

        self.get_logger().info('📸 Patrol Image Processor started')

    def check_gpu(self):
        """CUDA GPU 사용 가능 여부 확인"""
        try:
            # CuPy 또는 OpenCV CUDA 확인
            if cv2.cuda.getCudaEnabledDeviceCount() > 0:
                cv2.cuda.setDevice(0)
                return True
        except:
            pass
        return False

    def image_callback(self, msg):
        """카메라 이미지 수신"""
        try:
            self.last_image = self.bridge.imgmsg_to_cv2(msg, 'bgr8')

            # Waypoint 도착 시 캡처
            if self.capture_on_next and self.last_image is not None:
                self.capture_and_analyze(self.last_image)
                self.capture_on_next = False
        except Exception as e:
            self.get_logger().error(f'Image callback error: {e}')

    def waypoint_callback(self, msg):
        """Waypoint 도착 신호 수신"""
        waypoint_name = msg.data
        self.get_logger().info(f'📍 Arrived at {waypoint_name}')
        self.capture_on_next = True

    def capture_and_analyze(self, image):
        """이미지 저장 및 간단 분석"""
        timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
        filename = f'{self.save_dir}/patrol_{timestamp}.jpg'

        # GPU 가속 처리 (선택)
        if self.use_gpu:
            try:
                # GPU 메모리로 업로드
                gpu_frame = cv2.cuda_GpuMat()
                gpu_frame.upload(image)

                # GPU에서 Grayscale 변환
                gpu_gray = cv2.cuda.cvtColor(gpu_frame, cv2.COLOR_BGR2GRAY)

                # GPU에서 Gaussian Blur
                gpu_blur = cv2.cuda.gaussianBlur(gpu_gray, (5, 5), 1.0)

                # 결과를 CPU로 다운로드
                gray = gpu_blur.download()
            except Exception as e:
                self.get_logger().warn(f'GPU processing failed: {e}, falling back to CPU')
                gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
                gray = cv2.GaussianBlur(gray, (5, 5), 1.0)
        else:
            # CPU 처리
            gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
            gray = cv2.GaussianBlur(gray, (5, 5), 1.0)

        # 객체 유무 분석 (Contour 기반)
        edges = cv2.Canny(gray, 50, 150)
        contours, _ = cv2.findContours(
            edges, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

        # 일정 크기 이상의 객체 필터링
        significant_objects = [
            c for c in contours if cv2.contourArea(c) > 500
        ]

        # 결과 이미지에 표시
        result_img = image.copy()
        for c in significant_objects:
            x, y, w, h = cv2.boundingRect(c)
            cv2.rectangle(result_img, (x, y), (x + w, y + h), (0, 255, 0), 2)

        # 라벨 추가
        label = f'Objects: {len(significant_objects)}'
        cv2.putText(result_img, label, (10, 30),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 0), 2)

        # 이미지 저장
        cv2.imwrite(filename, result_img)

        # 결과 발행
        status_msg = String()
        status_msg.data = f'{timestamp}: {len(significant_objects)} objects'
        self.status_pub.publish(status_msg)

        self.motion_pub.publish(Bool(
            data=len(significant_objects) > 0))

        self.get_logger().info(
            f'💾 Saved: {filename} — {len(significant_objects)} objects')

    def destroy_node(self):
        self.get_logger().info('Image processor stopped')
        super().destroy_node()


def main(args=None):
    rclpy.init(args=args)
    node = PatrolImageProcessor()
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

---

## 20.2 CuPy 설치 (GPU 가속 선택)

CuPy는 NVIDIA GPU에서 NumPy 호환 배열 연산을 가능하게 하는 라이브러리입니다.

```bash
# Docker 컨테이너 내부에서
pip3 install cupy-cuda102

# CuPy 설치 확인
python3 -c "
import cupy as cp
print('CuPy version:', cp.__version__)
print('GPU:', cp.cuda.runtime.getDeviceProperties(0)['name'])
"
```

---

## 20.3 설치 (setup.py)

```python
# patrol_bot_nano/setup.py
entry_points={
    'console_scripts': [
        'image_processor = patrol_bot_nano.image_processor:main',
        'patrol_navigator = patrol_bot_nano.patrol_navigator:main',
    ],
},
```

---

## 20.4 성능 예상

| 작업 | CPU | GPU (CuPy/OpenCV CUDA) |
|------|-----|----------------------|
| Grayscale 변환 (640x480) | ~5ms | ~2ms |
| Gaussian Blur | ~8ms | ~3ms |
| Canny Edge | ~15ms | ~8ms |
| Contour 찾기 | ~10ms | CPU only |
| 이미지 저장 | ~20ms | ~20ms |
| **총 처리 시간** | **~58ms** | **~43ms** |

> GPU 가속 효과는 있지만, Maxwell 128-core의 한계로 큰 폭의 개선은 어렵습니다.
> 복잡한 딥러닝보다는 **기본 이미지 처리 가속**에 초점을 맞춥니다.

---

## 20.5 통합 실행

```bash
# 1. USB 카메라 실행
ros2 run v4l2_camera v4l2_camera_node

# 2. 이미지 프로세서
ros2 run patrol_bot_nano image_processor

# 3. Waypoint Navigator (Nav2 기반)
ros2 run patrol_bot_nano patrol_navigator

# 4. Remote PC에서 순찰 이미지 확인
rsync -avz jetson@jetson-nano:~/patrol_images/ ./patrol_images/
```

---

## 📝 연습 문제

1. OpenCV CUDA 함수 3개를 추가로 사용하여 이미지 처리 파이프라인을 확장하세요
2. CuPy를 사용한 GPU 행렬 연산 속도를 CPU와 비교하여 측정하세요
3. 캡처된 이미지에 타임스탬프와 GPS(Odometry) 위치를 오버레이하세요
4. 특정 Waypoint에서만 더 자세한 이미지 분석을 수행하는 로직을 추가하세요
5. GPU 메모리 사용량을 모니터링하고, 메모리 부족 상황을 처리하는 코드를 추가하세요
