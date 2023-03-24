---
layout: single
title: "다부처 과제 인수인계용 정리본 Hyper-parameter 정리하는 법"
author_profile: true
categories:
  - uvi_study
toc: true
toc_sticky: true
sidebar:
    nav: "sidebar-category"
---

## Hyper-parameter?
배회자 및 쓰러짐 결과에 대한 Hyper-parameter를 설정할 수 있는 방법을 알려주는 인수인계임.  
우리 연구실에서 바꿀 수 있는 Hyper-parameter는 크게 2종류로 되어 있음.
1. falldown의 bounding box ratio Threshold
2. wanderer의 tracking method select

## Falldown의 bounding box ratio Threshold
falldown의 threshold를 바꿀 수 있는 device는 edge, server 둘다 가능함

### edge module falldown hyper-parameter
`/workspace/detector/event/falldown/main.py`를 확인하면 111번 line ~ 121번 line에서 변경 가능함.
```
if int(info_['position']['w']) >= int(info_['position']['h']): #falldown
    if self.before_falldown_count[count] >= 55: 
        pass
    else:
        if int(info_['position']['y']) > 3 and int(info_['position']['y']) + int(info_['position']['h']) < 356:
            self.before_falldown_count[count] += 1
        else:
            if self.before_falldown_count[count] > 0:
                self.before_falldown_count[count] -= 1
elif  self.before_falldown_count[count] > 0: 
    self.before_falldown_count[count] -= 1
```
`if int(info_['position']['w']) >= int(info_['position']['h']):`에서 ratio 값을 변경하면 hyper-parameter 변경 가능
MLP를 사용한 ratio 추정에서 hyper-parameter를 변경하고자 하면, 131~139 line을 확인하면 됨.
```
if self.result_ratio>1.35:
    if self.before_falldown_count[count] >= 55:
        pass
    else:
        if int(info_['position']['y']) > 3 and int(info_['position']['y']) + int(info_['position']['h']) < 356:
            self.before_falldown_count[count] += 1
        else:
            if self.before_falldown_count[count] > 0:
                self.before_falldown_count[count] -= 1
```
여기서 `if self.result_ratio>1.35:`에서 값을 변경하면 됨

### server-module에서 falldown hyper-parameter 변경
`/workspace/detector/event/falldown/main.py`에서 68~70 line에 있는 ratio를 변경하면됨
```
bbox_width = detection_result[i]['position']['w']
bbox_height = detection_result[i]['position']['h']
bbox_ratio = bbox_width / bbox_height
```

### 추가적인 falldown Hyper-parameter
`falldown/main.py`를 보면 detection되는 사람 confidence를 수정할 수도 있음.

## Wanderer의 Tracking Method 변경
현재 yolov4, yolov7 둘다 변경 가능한데 각 모델마다 정확도가 다르기 때문에 hyper-parameter를 변경하면서 가장 좋은걸로 골라야함.

### edge 상황에서 tracking method yolov4, yolov7
`/workspace/config/params_yolov4.yml, test_params.yml`을 수정해야됨. 가장 먼저 `/workspace/config/params_yolov4.yml`에서 line 66~69 line으로 tracker method를 변경할 수 있음.
```
wanderer:
model_name: wanderer
score_threshold: 0.1
tracker_name: byte_tracker
```
line 26~39 line을 tracker parameter를 변경할 수 있음.
```
tracker:
    byte_tracker:
    tracker_name: byte_tracker
    score_threshold: 0.1
    track_threshold: 0.5
    track_buffer: 30
    match_threshold: 0.8
    min_box_area: 10
    frame_rate: 20
    sort_tracker:
    tracker_name: sort_tracker
    score_threshold: 0.5
    max_age: 2
    min_hits: 3
```
`test_params.yml`를 같이 변경해야함. line 29~42, line 60~63를 수정해야함.
```
tracker:
    byte_tracker:
    tracker_name: byte_tracker
    score_threshold: 0.1
    track_threshold: 0.5
    track_buffer: 30
    match_threshold: 0.8
    min_box_area: 10
    frame_rate: 20
    sort_tracker:
    tracker_name: sort_tracker
    score_threshold: 0.5
    max_age: 2
    min_hits: 3
```
```
wanderer:
model_name: wanderer
score_threshold: 0.1
tracker: sort_tracker
```

### server 상황에서 tracking method yolov7x
`/workspace/config/params.yml, test_params.yml`을 수정해야됨.  
가장 먼저 `/workspace/config/params.yml`에서 line 21~33 line, 65~68 line으로 tracker method를 변경할 수 있음.  
`21~33 line`  
```
byte_tracker:
  tracker_name: byte_tracker
  score_threshold: 0.1
  track_threshold: 0.5
  track_buffer: 30
  match_threshold: 0.8
  min_box_area: 10
  frame_rate: 10
sort_tracker:
  tracker_name: sort_tracker
  score_threshold: 0.5
  max_age: 2
  min_hits: 3
```
`65~68 line`
```
wanderer:
  model_name: wanderer
  score_threshold: 0.1
  tracker_name: sort_tracker
```