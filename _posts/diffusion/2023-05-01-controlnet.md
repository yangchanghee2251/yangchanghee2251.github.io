---
layout: single
title: "ControlNet Demo"
author_profile: true
categories:
  - diffusion
toc_label: "목차"
toc: true
toc_sticky: true
sidebar:
    nav: "sidebar-category"
---


## ControlNet
이번에 포스팅할 Diffusion Model은 [ControlNet](https://github.com/lllyasviel/ControlNet)이다.  
최근 포스팅이 뜸했는데 그 이유가 이번 ControlNet으로 연구를 진행하고 있어서 여러가지 검증을 진행하다보니 뜸해졌다.  
아무튼 ControlNet에 대한 간략한 설명과 Demo Code를 돌리는 방법을 설명하고자 한다.  
모델 구조는 다음과 같다.  
<img src="../post_images/controlnet.png" width="50%"  title="table1" alt=""/>  

[controlNet-arxiv](https://arxiv.org/abs/2302.05543)를 보면 굉장히 급하게 만든 느낌이 들긴하지만 Control을 넣어줌으로써 다양한 image를 생성하는 모델을 구현했다.  
기존의 diffusion model들은 prompt를 통해서 이미지를 생성한다. 하지만 이는 prompt에 굉장히 국한되고 내가 정말 원하는 task에 적용하기 힘든점이 있다.  
이를 Control(Scribble, Pose, Segmentation, Depth 등) 이라는 개념으로 조정(Control)할 수 있게끔 만든 모델이라고 생각하면 편할 것 같다.

## ControlNet 간단 설명
ControlNet인 위의 모델 구조를 보면 알다시피 Stable diffusion을 baseline으로 ControlNet이라는 Module을 만들어서 Decoder 단에 넣는 개념이다.  
여기서 Zero Convolution을 넣어서 학습을 시작하는데 이 개념이 굉장히 머리가 좋다라는 느낌을 받았다.  
그 이유는 Zero Convolution을 진행하면서 처음 학습을 시작할 때 Stable Diffusion에는 어떠한 방해도 하지 않고 시작하기 때문에 안정적으로 훈련이 가능하다는 설명을 진행하고 있다.  
(Control을 하려면 시작에 대한 훈련이 안정적으로 진행이 되야하기 때문에 이러한 아이디어를 적용하지 않았나 싶다.)

## ControlNet Demo
ControlNet에 대한 Demo는 생각보다 쉽게 나와있다.

### hugging face
현재 v1에 대한 hugging face는 더이상 지원하지 않고 v1.1에 대한 버젼만 지원하고 있다. [hugging_face_demo](https://huggingface.co/lllyasviel/ControlNet-v1-1)를 들어가면 쉽게 demo를 진행할 수 있다.

### code demo
`gradio_canny.py, gradio_depth.py, gradio_scribble.py, ...`등 다양하게 사용이 가능하다.  
이를 사용하기 위해서는 
```
conda env create -f environment.yaml
conda activate control
```
이렇게 conda 환경을 구축을 해야한다.  
controlNet에서 기본적으로 구현되어 있는 code가 gr을 사용해서 진행을 하는데 이를 사용하기 싫으면 아래의 코드를 사용하면 된다.  
Ex) graido_scribble2image.py
```
from share import *
import config

import cv2
import einops
import gradio as gr
import numpy as np
import torch
import random

from pytorch_lightning import seed_everything
from annotator.util import resize_image, HWC3
from cldm.model import create_model, load_state_dict
from cldm.ddim_hacked import DDIMSampler


model = create_model('./models/cldm_v15.yaml').cpu()
model.load_state_dict(load_state_dict('./models/control_sd15_scribble.pth', location='cuda'))
model = model.cuda()
ddim_sampler = DDIMSampler(model)


def process(input_image, prompt, a_prompt, n_prompt, num_samples, image_resolution, ddim_steps, guess_mode, strength, scale, seed, eta):
    with torch.no_grad():
        img = resize_image(HWC3(input_image), image_resolution)
        H, W, C = img.shape

        detected_map = np.zeros_like(img, dtype=np.uint8)
        detected_map[np.min(img, axis=2) < 127] = 255

        control = torch.from_numpy(detected_map.copy()).float().cuda() / 255.0
        control = torch.stack([control for _ in range(num_samples)], dim=0)
        control = einops.rearrange(control, 'b h w c -> b c h w').clone()

        if seed == -1:
            seed = random.randint(0, 65535)
        seed_everything(seed)

        if config.save_memory:
            model.low_vram_shift(is_diffusing=False)

        cond = {"c_concat": [control], "c_crossattn": [model.get_learned_conditioning([prompt + ', ' + a_prompt] * num_samples)]}
        un_cond = {"c_concat": None if guess_mode else [control], "c_crossattn": [model.get_learned_conditioning([n_prompt] * num_samples)]}
        shape = (4, H // 8, W // 8)

        if config.save_memory:
            model.low_vram_shift(is_diffusing=True)

        model.control_scales = [strength * (0.825 ** float(12 - i)) for i in range(13)] if guess_mode else ([strength] * 13)  # Magic number. IDK why. Perhaps because 0.825**12<0.01 but 0.826**12>0.01
        samples, intermediates = ddim_sampler.sample(ddim_steps, num_samples,
                                                     shape, cond, verbose=False, eta=eta,
                                                     unconditional_guidance_scale=scale,
                                                     unconditional_conditioning=un_cond)

        if config.save_memory:
            model.low_vram_shift(is_diffusing=False)

        x_samples = model.decode_first_stage(samples)
        x_samples = (einops.rearrange(x_samples, 'b c h w -> b h w c') * 127.5 + 127.5).cpu().numpy().clip(0, 255).astype(np.uint8)

        results = [x_samples[i] for i in range(num_samples)]
    return [255 - detected_map] + results

input_image=cv2.imread("test")
prompt="test"
num_samples=3
seed =12345
det = "Scribble_HED"
image_resolution =512
strength =1.0
guess_mode =False
detect_resolution =512
ddim_steps =20
scale = 9.0 
eta=1.0
value = 1.0
a_prompt = 'best quality, extremely detailed, realistic,person'
n_prompt = 'longbody, lowres, bad anatomy, bad hands, missing fingers, extra digit, fewer digits, cropped, worst quality, low quality'

result=process(input_image, prompt, a_prompt, n_prompt, num_samples, image_resolution, detect_resolution, ddim_steps, guess_mode, strength, scale, seed, eta)
```