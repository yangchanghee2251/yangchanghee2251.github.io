---
layout: single
title: "PP_YOLOE Demo & Evaluation 하는 법"
author_profile: true
categories:
  - tracking

toc_label: "목차"
toc: true
toc_sticky: true
sidebar:
    nav: "sidebar-category"
---

## PP-YOLOE?
[MOT16 성능](https://paperswithcode.com/sota/multi-object-tracking-on-mot16)를 확인하면 MOT16에서 가장 성능이 높은 알고리즘임. Demo를 한번 돌려보기 위해서 진행해보기로 하자.  
[PP-YOLOE 논문](https://arxiv.org/pdf/2203.16250v3.pdf)을 보면 PP-YOLOE의 Arxiv 논문이다. Conference등에 등록된 논문은 아닌 것 같지만 그래도 성능이 높기 때문에 한번 시도해보자.  
[Officail-Code (PaddlePaddle)](https://github.com/PaddlePaddle/PaddleDetection) 공식 코드는 PaddlePaddle로 짯다. 하지만 난 pytorch를 사용할 거기 떄문에 Pytorch Code로 작성한 코드는 [여기](https://github.com/Nioolek/PPYOLOE_pytorch)에 있다.  
**하지만 바이두 아이디가 없는 이상 위에 방법은 할 수 없다.** 따라서 우회 방법으로 [mmyolo](https://github.com/open-mmlab/mmyolo/tree/main/configs/ppyoloe)를 사용해보고자 한다.

## 내 실험 환경 (바이두 아이디 있는 경우)
OS는 Linux 20.04 LTS에 GPU는 RTX 3090 1대이다. cuda버젼은 11.1~11.6 까지 전부 깔아놨지만 지금은 cuda 11.6으로 실험해보고자 한다. 아나콘다 가상환경으로 진행할 예정이다.  
대충 pytorch code github에서 확인해본 결과 python이 3.6이상이여야 되서 python은 3.8로 해볼 예정이다.
```
conda create -n PPYOLOE python=3.8
conda activate PPYOLOE
```
설치가 완료가 되고 나는 cudda 11.6으로 진행할거기 때문에 [공식 pytorch 홈페이지](https://pytorch.org/get-started/previous-versions/) 여기서 cuda 11.6 버젼을 맞춰서 설치할거다.
```
pip install torch==1.13.0+cu116 torchvision==0.14.0+cu116 torchaudio==0.13.0 --extra-index-url https://download.pytorch.org/whl/cu116
```
완료되면 나오는 코드
```
Installing collected packages: urllib3, typing-extensions, pillow, numpy, idna, charset-normalizer, torch, requests, torchvision, torchaudio
Successfully installed charset-normalizer-3.1.0 idna-3.4 numpy-1.21.6 pillow-9.4.0 requests-2.28.2 torch-1.13.0+cu116 torchaudio-0.13.0+cu116 torchvision-0.14.0+cu116 typing-extensions-4.5.0 urllib3-1.26.15
```
pytorch 버젼 github 에서 말한대로 `git clone, pip3 install -v -e`를 해보자.
```
git clone git@github.com:Nioolek/PPYOLOE_pytorch.git
cd PPYOLOE_pytorch
pip3 install -v -e .  # or  python3 setup.py develop
```

## Weight file 다운로드 (바이두 아이디 있는 경우)
현재 [공식 pytorch 홈페이지](https://pytorch.org/get-started/previous-versions/)여기서는 바이두 아이디가 있는 사람만 weight file을 가져갈 수 있다...(중국인만 가능..) 그래서 좀 위회해서 하는 법을 찾아본 결과 바이두 아이디 위회해서 다운로드가 가능하다. 하지만 난 바이두 아이디를 만들지 않을거기 때문에 다른 방법을 생각해보려고 한다.

## mmyolo 사용하는 방법 (바이두 아이디 없는 경우) [실험 환경]
OS는 Linux 20.04 LTS에 GPU는 RTX 3090 1대이다. mmyolo는 cuda 11.3로 실험해야 하고 아나콘다 가상환경으로 진행할 예정이다.  
대충 pytorch code github에서 확인해본 결과 python이 3.8이여야 되서 python은 3.8로 해볼 예정이다.
cuda 버젼 바꾸는 방법은 [여기]()를 확인하면 된다.