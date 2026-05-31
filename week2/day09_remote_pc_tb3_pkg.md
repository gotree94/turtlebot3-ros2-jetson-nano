# Day 9 — Remote PC TurtleBot3 패키지

> **RPi 버전 Day 9와 내용이 동일합니다.**

Remote PC에서 RViz2 시각화 및 텔레옵을 위한 패키지를 설치합니다.

```bash
# Remote PC (Ubuntu 22.04)
sudo apt install -y ros-humble-turtlebot3*

export TURTLEBOT3_MODEL=burger
export ROS_DOMAIN_ID=30
echo 'export TURTLEBOT3_MODEL=burger' >> ~/.bashrc
echo 'export ROS_DOMAIN_ID=30' >> ~/.bashrc
```

> **전체 내용:** `../../turtlebot3-ros2-curriculum/week2/day09_remote_pc_tb3_pkg.md` 참조
