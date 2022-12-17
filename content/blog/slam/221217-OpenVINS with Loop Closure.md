---
title: "OpenVINS with Loop Closure"
date: 2022-12-17 22:51:00 -0400
category: slam
draft: false
---

## **How VINS-Fusion Loop Closure Works**

1. `카메라의 intrinsic parameter`와 `CAM_to_IMU에 대한 extrinsic parameter`를 subscribe
    - OpenVINS가 Online으로 수행되는 경우 두 파라미터는 시간이 지남에 따라 변경됨
    - 최신 calibration된 값을 얻기 위해, 두 토픽을 subscribe
        - `/ov_msckf/loop_intrinsics`
        - `/ov_msckf/loop_extrinsic`

2. `pose_graph_node.cpp`에서 **image**, **pointcloud**, **odometry**에 해당하는 message들을 OpenVINS로 부터 가져 옴
    - `/camera/infra1/image_rect_raw`
    - `/ov_msckf/loop_feats`: global frame 내에서 모든 3D feature들
        - raw uv coordinates, normalized coordinates, feature id을 포함
    - `/ov_msckf/loop_pose`

3. keyframe을 pose graph에 추가
    - 충분히 이동한 경우에 대해서만 keyframe을 pose graph에 추가함

4. keyframe의 정보들을 처리
    - 모든 global 3D position과 그에 상응하는 uv 좌표들을 저장
    - 모든 feature들에 대해 brief descriptor를 추출

5. loop closure 검출 시도
    - `pose_graph.cpp`에서 `addKeyFrame()` 함수는 keyframe을 찾기 위해 `detectLoop()`를 호출
        - 과거의 모든 keyframe이 포함된 DBoW2 데이터베이스를 쿼리
    - return된 상위 n개의 match에서, 모든 임계 값을 통과하는 가장 오래된 match를 선택
        - 가장 오래된 match는 가장 작은 id를 가지며, 유효한 PnP pose 얻을 수 있어야 함

6. pose graph optimization 수행
    - 일치하는 keyframe을 찾은 경우, 그래프를 최적화
    - loop closure에 대한 keyframe은 `optimize_buf`에 추가되고, `optimize4DoF()` thread에서 처리됨
    - 모든 keyframe들을 매개변수로 추가
    - keyframe에 loop closure가 있는 경우, 엣지로 추가
    - 항상 마지막 frame과 현재 frame 사이의 상관관계를 추가 (OpenVINS odometry 기반)
    - 최적화 후 new keyframe이 **얼마나 수정되어야 하는지** 정도를 계산
        - `yaw _drift`, `r_drift`, `t_drift` 등등

7. pose를 publish
    - pose graph는 항상 추정치보다 **"지연(lags)"**된다는 점에 유의해야 함
    - 현재 state는 마지막으로 추가된 keyframe으로 회전
    - 그런 다음, 원래 keyframe에 대한 "수정"이 현재 state에 적용
    - 수정된 pose는 다시 publish됨
        - `vio_callback()`

## **Identified Limitations**

- 그래프에는 일정한 가중치가 있으므로, 하나의 잘못된 loop closure가 전체 추정치를 손상시킬 수 있음
- keyframe은 한 번만 loop closure될 수 있으므로, 지속적으로 수행되지 않음
- bag-of-word 조회가 실패할 수 있음
- 사용된 feature는 절대 개선되거나 다시 최적화되지 않으며, **true**로 간주됨
- 시스템의 튜닝이 어렵고, 제대로 튜닝되지 않으면 성능이 저하될 수 있음
