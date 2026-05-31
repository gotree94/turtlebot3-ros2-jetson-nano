# Day 13 — TurtleBot3 URDF & RViz2 시각화

> **RPi 버전 Day 13과 내용이 동일합니다.**
>
> RViz2는 Remote PC에서 실행하며, URDF 모델과 TF 트리를 확인합니다.

---

## 실행 구성

```
Jetson Nano (네이티브)          Remote PC
─────────────────────          ──────────
robot_state_publisher ──/tf──►  RViz2
  (URDF 로드)                   - RobotModel
  /joint_states                 - TF 표시
                                - 3D 모델 렌더링
```

> **전체 내용:** `../../turtlebot3-ros2-curriculum/week2/day13_rviz2_urdf.md` 참조
>
> 실습: URDF 로드, TF 트리 확인, Link/Joint 시각화 완전 동일
