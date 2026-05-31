# Day 1 — Jetson Nano 4GB OS 설치 & 스토리지 설정

> **목표:** Jetson Nano 4GB (SD 카드 모듈, eMMC 없음)에 JetPack 4.6.1을 설치하고,
> USB SSD 부팅까지 설정한다. 개발용 모듈과 양산용 모듈의 차이를 이해한다.

---

## 1.1 Jetson Nano 모듈 이해하기

### SD 카드 모듈 (P3448-0000) — 이 커리큘럼의 타겟

```
┌──────────────────────────────────────────────────┐
│  Jetson Nano SD Card Module (P3448-0000)         │
│  ─── "개발용 / eMMC 없는 버전"                    │
│                                                   │
│  • GPU: NVIDIA Maxwell 128 CUDA cores             │
│  • CPU: Quad ARM Cortex-A57 @ 1.43 GHz           │
│  • RAM: 4GB LPDDR4 (25.6 GB/s)                   │
│  • eMMC: ❌ 없음 (온보드 저장소 제로)               │
│  • SD 카드 슬롯: ✅ 모듈 자체 내장                 │
│  • 부트로더 저장: QSPI-NOR flash (SPI)             │
│  • 용도: 학습, 개발, 프로토타이핑                   │
│  • Developer Kit 번들: ~$99 (캐리어 보드 포함)      │
└──────────────────────────────────────────────────┘
```

**이 모듈의 핵심 특징:**
- **저장 장치 없음** → OS를 반드시 SD 카드 또는 USB SSD에 설치
- Developer Kit에 포함된 캐리어 보드(P3449)에 장착되어 출시
- SD 카드에서 직접 부팅하거나, JetPack 4.5+부터 USB SSD 부팅 가능
- 대부분의 온라인에서 "Jetson Nano Developer Kit"으로 판매되는 버전

### eMMC 모듈 (P3448-0001/0002) — 양산용

```
┌──────────────────────────────────────────────────┐
│  Jetson Nano eMMC Module (P3448-0001/0002)       │
│  ─── "양산용 / 산업용"                            │
│                                                   │
│  • GPU/CPU/RAM: SD 카드 모듈과 동일               │
│  • eMMC: ✅ 16GB eMMC 5.1 (온보드)                │
│  • SD 카드 슬롯: ❌ 모듈에 없음                     │
│  • 부트로더 저장: eMMC 자체                        │
│  • 용도: 상용 제품, 산업용 임베디드 시스템           │
│  • 가격: ~$129 (모듈 단품, 캐리어 보드 미포함)      │
│  • 보증: 확장 보증, 산업용 온도 범위(-25°C~97°C)    │
└──────────────────────────────────────────────────┘
```

**양산용 모듈의 특징:**
- 16GB eMMC가 온보드에 직접 납땜되어 있어 **SD 카드 불필요**
- 자체 캐리어 보드를 설계하는 전문 기업 대상 (제품 개발사)
- SD 카드 슬롯이 없으므로 SDK Manager로 USB 리커버리 모드만 가능
- 더 넓은 온도 범위와 확장 보증으로 산업용 신뢰성 확보

### ⚠️ Developer Kit (개발용 키트) vs Production Module (양산용 모듈)

| 구분 | Developer Kit (개발용 키트) | Production Module (양산용 모듈) |
|------|--------------------------|-------------------------------|
| **구성** | SD 카드 모듈 + 캐리어 보드 + 방열판 | eMMC 모듈 단품 |
| **저장소** | microSD 카드 (별도 구매) | 16GB eMMC 내장 |
| **OS 설치** | SD 카드 이미지 또는 SDK Manager | SDK Manager (USB 리커버리) |
| **부팅** | SD 카드 → USB SSD | eMMC 직접 부팅 |
| **확장 저장소** | SD 카드 + USB | microSD 슬롯 없음, USB만 가능 |
| **대상 고객** | 교육, 개발자, 메이커, 연구자 | OEM, 제품 개발사, 산업용 |
| **필요 추가 부품** | SD 카드, USB SSD(권장), 전원, HDMI 케이블 | 캐리어 보드(별도 설계/구매), 전원 |
| **가격** | ~$99 (완전한 키트) | ~$129 (모듈만) |

---

## 1.2 준비물

- NVIDIA Jetson Nano Developer Kit (SD 카드 모듈 버전, 4GB)
- **(권장) USB 3.0 SSD** 128GB+ (OS 설치 및 ROS2 워크스페이스용)
- MicroSD 카드 64GB+ (Class 10 U3/A2)
- MicroSD 리더기
- **DC 배럴잭 전원 어댑터 5V 4A** — **Micro-USB로는 부족합니다**
- 점퍼 와이퍼 (J48 커넥터, DC 전원 활성화용)
- **호스트 PC (x86_64)** — Ubuntu 20.04/22.04 권장 (SDK Manager 실행)
- HDMI 모니터 + USB 키보드/마우스
- USB 케이블 (Micro-B, 리커버리 모드 연결용)
- (선택) 이더넷 케이블

> ⚠️ **전원 주의:** Jetson Nano는 Micro-USB(5V 2A, 10W)와 DC 배럴잭(5V 4A, 20W)을
> 모두 지원하지만, USB SSD + ROS2 + TurtleBot3 동시 사용 시
> **반드시 DC 배럴잭 5V 4A 전원**을 사용해야 안정적입니다.
> J48 핀에 점퍼를 연결하여 DC 전원을 활성화해야 합니다.

---

## 1.3 J48 점퍼 설정 (DC 전원 활성화)

Jetson Nano Developer Kit은 기본적으로 Micro-USB로 전원을 공급받습니다.
DC 배럴잭을 사용하려면 보드 위의 J48 핀을 쇼트해야 합니다.

```
Jetson Nano 보드 레이아웃:

            ┌──────────────────────────────┐
            │  J48 (DC 전원 선택)           │
            │  ● ●  ← 이 두 핀을 쇼트       │
            │  (점퍼 와이퍼 연결)            │
            │                              │
  [DC IN]──┤           [Micro-USB]         │
  (5V 4A)  │                    (5V 2A)    │
            └──────────────────────────────┘
```

1. 보드에서 J48 핀 찾기 (USB 포트 옆, 2핀 헤더)
2. 점퍼 와이퍼 또는 핀셋으로 두 핀 쇼트
3. 확인: J48 쇼트 시 DC 배럴잭 우선, 미쇼트 시 Micro-USB 우선

---

## 1.4 USB SSD 준비 (권장)

SD 카드는 ROS2 패키지 빌드 시 I/O 병목이 심각하므로 **USB SSD 사용을 강력 권장**합니다.

```bash
# 호스트 PC에서 USB SSD를 ext4로 포맷
sudo fdisk /dev/sdX   # X는 USB SSD 장치명
# → g (GPT 파티션 테이블)
# → n (새 파티션)
# → w (쓰기)

sudo mkfs.ext4 /dev/sdX1
sudo e2label /dev/sdX1 JETSON_ROOT
```

---

## 1.5 JetPack 4.6.1 설치

### SDK Manager로 설치 (권장)

**호스트 PC (x86_64 Ubuntu)에서 실행:**

```bash
# 1. SDK Manager 다운로드 & 설치
# https://developer.nvidia.com/sdk-manager
sudo apt install ./sdkmanager_[version].deb

# 2. Jetson Nano 준비
#  - 전원 OFF
#  - Micro-USB 케이블로 호스트 PC 연결
#  - J48 점퍼 설정 확인 (DC 전원 사용 시)
#  - 전원 연결
#  - 리커버리 버튼(FORCE RECOVERY) 누른 상태
#  - 리셋 버튼(RESET) 1초 누름 → 리커버리 버튼 해제
#  - 호스트 PC에서 확인:
lsusb | grep -i nvidia
# → "NVidia Corp. APX" 표시되어야 함

# 3. SDK Manager 실행
sdkmanager

# 4. 설정:
#  - Product: Jetson Nano
#  - JetPack Version: 4.6.1 (최종 지원 버전)
#  - Storage: SD Card (SD 카드 모듈인 경우)
#  - 구성 요소 선택:
#    ✅ Jetson Linux (BSP)
#    ✅ CUDA 10.2
#    ✅ cuDNN 8.x
#    ✅ TensorRT 8.x
#    ✅ OpenCV for Jetson (4.1.1)
#    ✅ Multimedia API (L4T)
#  - Install → 약 20~40분 소요
```

### 설치 완료 후

```bash
# 1. SD 카드/SD 카드로 부팅
# 2. 초기 OOBE 설정:
#    - 언어: English
#    - 사용자: jetson / [비밀번호]
#    - 호스트명: jetson-nano
# 3. 시스템 업데이트
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y
sudo reboot
```

---

## 1.6 USB SSD 부팅 설정 (JetPack 4.5+)

JetPack 4.5부터 QSPI-NOR 부트로더 업데이트로 USB 부팅이 가능해졌습니다.

### Step 1: QSPI-NOR 확인

```bash
# QSPI-NOR flash가 업데이트되었는지 확인
# (SDK Manager로 설치 시 자동 업데이트)
ls /dev/mtd*
# /dev/mtd0 등이 표시되면 정상
```

### Step 2: USB SSD 마운트

```bash
# USB SSD 연결 확인
lsblk
# sda (또는 sdb)로 표시

# 마운트
sudo mkdir -p /mnt/usb
sudo mount /dev/sda1 /mnt/usb
```

### Step 3: RootFS 복사

```bash
# rootOnUSB 스크립트 사용 (또는 수동 rsync)
git clone https://github.com/JetsonHacksNano/rootOnUSB.git
cd rootOnUSB

# USB SSD로 전체 시스템 복사
./copyRootToUSB.sh -p /dev/sda1

# PARTUUID 확인 (더 안정적인 부팅)
./partUUID.sh
```

### Step 4: 부트로더 수정

```bash
# extlinux.conf 수정
sudo cp /boot/extlinux/extlinux.conf /boot/extlinux/extlinux.conf.backup
sudo nano /boot/extlinux/extlinux.conf
```

**수정 내용:**
```
LABEL primary
  LINUX /boot/Image
  INITRD /boot/initrd
  APPEND ${cbootargs} root=/dev/sda1 rw rootwait
#                           ↑ SD 카드(/dev/mmcblk0p1) → USB(/dev/sda1)로 변경
```

### Step 5: USB SSD 부팅 테스트

```bash
# SD 카드를 제거한 후 재부팅
sudo shutdown -h now
# SD 카드 제거
# USB SSD만 연결한 상태로 전원 ON
# USB SSD에서 부팅되는지 확인
df -h /
# /dev/sda1 → 성공!
```

### USB 부팅 성능 비교

```bash
# SD 카드 읽기 속도
sudo hdparm -t /dev/mmcblk0
# ~87 MB/s

# USB SSD 읽기 속도
sudo hdparm -t /dev/sda
# ~367 MB/s (약 4배 향상)
```

---

## 1.7 초기 시스템 설정

```bash
# 필수 도구 설치
sudo apt update
sudo apt install -y \
  build-essential \
  cmake \
  git \
  curl \
  wget \
  vim \
  nano \
  htop \
  tree \
  net-tools \
  openssh-server \
  python3-pip \
  python3-venv

# SSH 활성화
sudo systemctl enable --now sshd

# WiFi 설정 (M.2 WiFi 모듈 장착 시)
nmcli dev wifi list
sudo nmcli dev wifi connect "WiFiSSID" password "WiFiPassword"

# 호스트 PC에서 SSH 접속 테스트
# ssh jetson@jetson-nano.local
```

---

## 1.8 CUDA & GPU 확인

```bash
# CUDA 버전 확인 (10.2)
nvcc --version

# GPU 정보
cat /proc/device-tree/model
# NVIDIA Jetson Nano Developer Kit

# GPU 실시간 모니터링
sudo apt install -y python3-pip
sudo pip3 install jetson-stats
jtop
```

---

## 1.9 전원 모드 설정

```bash
# 현재 전원 모드 확인
sudo nvpmodel -q

# 10W 모드 (최대 성능)
sudo nvpmodel -m 0

# 5W 모드 (절전)
sudo nvpmodel -m 1

# GPU 클럭 확인
sudo tegrastats | grep "GR3D"
```

---

## 1.10 확인 사항 체크리스트

- [ ] `cat /proc/device-tree/model` → Jetson Nano
- [ ] `nvcc --version` → CUDA 10.2
- [ ] `free -h` → 4GB RAM
- [ ] `df -h` → USB SSD 또는 SD 카드 충분한 공간
- [ ] `jtop` → GPU/CPU 정상 동작
- [ ] `sudo nvpmodel -q` → MAXN (10W) 모드
- [ ] `hostname -I` → IP 확인
- [ ] SSH 접속 가능 (호스트 PC에서)

---

## 📝 연습 문제

1. 자신의 Jetson Nano가 SD 카드 모듈인지 eMMC 모듈인지 확인하는 방법을 설명하세요
2. Developer Kit과 Production Module의 차이점 3가지를 쓰세요
3. USB SSD 부팅이 SD 카드 부팅보다 ROS2 개발에 유리한 이유를 설명하세요
4. J48 점퍼의 역할과 설정 방법을 설명하세요
5. `jtop`을 설치하고 Jetson Nano의 CPU/GPU 온도를 확인하세요
6. SD 카드와 USB SSD의 읽기 속도를 측정하고 비교하세요

---

## ⚠️ 트러블슈팅

| 문제 | 해결 방법 |
|------|----------|
| SDK Manager가 Jetson 감지 못함 | 리커버리 모드 재확인, Micro-USB 케이블 다른 포트 |
| USB SSD 부팅 안 됨 | QSPI-NOR 업데이트 필요 (SDK Manager 재설치) |
| 부팅 후 USB SSD 미인식 | `lsblk`, `fdisk -l`로 확인, `extlinux.conf` 재확인 |
| 전원 불안정 | DC 배럴잭 5V 4A 사용, J48 점퍼 확인 |
| CUDA 미인식 | `dpkg -l | grep cuda` 확인, SDK Manager 재실행 |
| SD 카드 모듈 vs eMMC 모듈 확인 | `lsblk`로 mmcblk0 eMMC 존재 여부 확인 |
| Jetson Nano 2GB vs 4GB | `free -h`로 메모리 확인, `cat /proc/meminfo` |
