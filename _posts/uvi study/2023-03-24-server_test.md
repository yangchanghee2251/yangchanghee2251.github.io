---
layout: single
title: "다부처 인수인계용 정리본 edge-analysis-module 테스트"
author_profile: true
categories:
  - uvi_study
toc_label: "목차"
toc: true
toc_sticky: true
sidebar:
    nav: "sidebar-category"
---

**edge-analysis-module Test**

## yolov4 모델 받기
`/workspace`상에서 아래 코드를 진행하면 됨.
```
sh scripts/download_libdarknet-cuda11.1-RTX3090.sh #(option)
sh scripts/download_models.sh
```

## edge-analysis-module Test
```
python3 test/test_video_all_model_batch.py --video_path=media/videos/total_videos/360p/mix_09_360p.mp4
```
edge 와 다르게 바로 테스트가 가능함.
```
[2023-03-24 17:38:00.379360] -     INFO: frame number:   5670/5751      / timestamp: 0:03:09.000[2023-03-24 17:38:00.503136] -     INFO: processing time:
                                                object detection(yolov7x) - total: 14.324152    / average: 0.002491     / max: 0.010604  / min: 0.007097
                                                pose estimation(alphapose) - total: 36.276311   / average: 0.009484     / max: 0.017312  / min: 0.000138
                                                tracker
                                                        byte_tracker - total: 1.519664  / average: 0.000264     / max: 0.002947  / min: 0.000112
                                                        sort_tracker - total: 1.083381  / average: 0.000188     / max: 0.001863  / min: 0.000082
                                                event detection
                                                        assault - total: 10.815698      / average: 0.001881     / max: 0.103455  / min: 0.000197
                                                        falldown - total: 0.121052      / average: 0.000021     / max: 0.000229  / min: 0.000015
                                                        kidnapping - total: 0.155306    / average: 0.000027     / max: 0.009001  / min: 0.000015
                                                        tailing - total: 9.186164       / average: 0.001597     / max: 0.038347  / min: 0.000005
                                                        wanderer - total: 9.498019      / average: 0.001652     / max: 0.045541  / min: 0.000007
                                                total process time: 90.316573
[2023-03-24 17:38:00.512059] -     INFO: event result file is successfully extracted.(path: results/video/mix_09_360p_server_10fps_230324.csv)
[2023-03-24 17:38:00.512335] -     INFO: sequence result file is successfully extracted.(path: results/video/mix_09_360p_server_10fps_230324.json)
[2023-03-24 17:38:26.886587] -     INFO: frame number:   1889/1890 - {'assault': False, 'falldown': False, 'kidnapping': False, 'tailing': False, 'wanderer': False}
[2023-03-24 17:38:26.996674] -     INFO: sequence result video is successfully generated(path: results/video/mix_09_360p_server_10fps_230324.mp4)
```
완료 됬을 때 결과 창

## 다부처 결과 확인하는 URL
[다부처 결과 확인 서버](mlwyberns.sogang.ac.kr:8777)를 접속하면 다부처 결과를 확인할 수 있음.