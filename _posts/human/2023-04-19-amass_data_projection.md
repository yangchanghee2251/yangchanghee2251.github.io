---
layout: single
title: "Amass dataset 2D Projection"
author_profile: true
categories:
  - human
toc_label: "목차"
toc: true
toc_sticky: true
sidebar:
    nav: "sidebar-category"
---


## Amass Dataset
[Amass](https://amass.is.tue.mpg.de/index.html)에서 제공하는 point cloud로 smpl registration을 거쳐 나온 parameter들을 모아둔 dataset이다.  
[Amass Download](https://amass.is.tue.mpg.de/download.php)에서 다양한 Dataset을 받을 수 있는데 SMPL+H G, SMPL-X G, SMPL-X N등이 있다. SMPL+H G는 SMPLH model을 거쳐서 나온 결과이며 G는 male or female의 정보를 가지고 있음을 보여준다. SMPL-X N은 SMPLX model을 거쳐 나온 parameter이며 N은 neutral로 나온 결과이다.  
나는 여기서 SMPL+H G Dataset들을 2D Image projection 시켜보고자 한다.

### requirements
기본적으로 T2M-GPT의 requirements상에서 진행했으며 다음과 같다.
```
torch==1.12.1+cu113 torchvision==0.13.1+cu113 torchaudio==0.12.1 --extra-index-url https://download.pytorch.org/whl/cu113
absl-py==0.13.0
backcall==0.2.0
cachetools==4.2.2
charset-normalizer==2.0.4
chumpy==0.70
cycler==0.10.0
decorator==5.0.9
google-auth==1.35.0
google-auth-oauthlib==0.4.5
grpcio==1.39.0
idna==3.2
imageio==2.9.0
ipdb==0.13.9
ipython==7.26.0
ipython-genutils==0.2.0
jedi==0.18.0
joblib==1.0.1
kiwisolver==1.3.1
markdown==3.3.4
matplotlib==3.4.3
matplotlib-inline==0.1.2
oauthlib==3.1.1
pandas==1.3.2
parso==0.8.2
pexpect==4.8.0
pickleshare==0.7.5
prompt-toolkit==3.0.20
protobuf==3.17.3
ptyprocess==0.7.0
pyasn1==0.4.8
pyasn1-modules==0.2.8
pygments==2.10.0
pyparsing==2.4.7
python-dateutil==2.8.2
pytz==2021.1
pyyaml==5.4.1
requests==2.26.0
requests-oauthlib==1.3.0
rsa==4.7.2
scikit-learn==0.24.2
scipy==1.7.1
sklearn==0.0
smplx==0.1.28
tensorboard==2.6.0
tensorboard-data-server==0.6.1
tensorboard-plugin-wit==1.8.0
threadpoolctl==2.2.0
toml==0.10.2
tqdm==4.62.2
traitlets==5.0.5
urllib3==1.26.6
wcwidth==0.2.5
werkzeug==2.0.1
git+https://github.com/openai/CLIP.git
git+https://github.com/nghorbani/human_body_prior
gdown
moviepy
```

너무 많은 requirements라면 아래만 사용해도 될 것으로 예상된다.
```
torch==1.12.1+cu113 torchvision==0.13.1+cu113 torchaudio==0.12.1 --extra-index-url https://download.pytorch.org/whl/cu113
matplotlib==3.4.3
matplotlib-inline==0.1.2
smplx==0.1.28
tqdm==4.62.2
git+https://github.com/nghorbani/human_body_prior
```
### 3D Pose preprocessing
HumanML3D에서 진행한 raw_pose_processing의 Code를 baseline으로 진행했다.  
Amass dataset의 디렉토리 path는 다음과 같다.
```
|amass_data
| - ACCAD
| - - Female1General_c3d
| - - - A1 - Stand_poses.npz
| - - - A2 - Sway t2_poses.npz
| ...
| - - Female1Gestures_c3d
| ...
| - BioMotionLab_NTroje
| ...
```
가장 먼저 필요한 package들을 선언해준다.
```
%load_ext autoreload
%autoreload 2
%matplotlib notebook
%matplotlib inline

import sys, os
import torch
import numpy as np
import matplotlib
import matplotlib.pyplot as plt
from tqdm import tqdm



from human_body_prior.tools.omni_tools import copy2cpu as c2c

os.environ['PYOPENGL_PLATFORM'] = 'egl'

# Choose the device to run the body model on.
comp_device = torch.device("cuda:2" if torch.cuda.is_available() else "cpu")
```
그 후에 human_body_prior를 이용해 smpl model을 load해준다.
```
from human_body_prior.body_model.body_model import BodyModel

male_bm_path = './body_models/smplh/male/model.npz'
male_dmpl_path = './body_models/dmpls/male/model.npz'

female_bm_path = './body_models/smplh/female/model.npz'
female_dmpl_path = './body_models/dmpls/female/model.npz'

num_betas = 10 # number of body parameters
num_dmpls = 8 # number of DMPL parameters

male_bm = BodyModel(bm_fname=male_bm_path, num_betas=num_betas, num_dmpls=num_dmpls, dmpl_fname=male_dmpl_path).to(comp_device)
faces = c2c(male_bm.f)

female_bm = BodyModel(bm_fname=female_bm_path, num_betas=num_betas, num_dmpls=num_dmpls, dmpl_fname=female_dmpl_path).to(comp_device)
```
female, male model을 로드를 해준 후에 다운받았던 amass dataset들의 path를 저장해준다.
```
paths = []
folders = []
dataset_names = []
for root, dirs, files in os.walk('./amass_data'):
#     print(root, dirs, files)
#     for folder in dirs:
#         folders.append(os.path.join(root, folder))
    folders.append(root)
    for name in files:
        dataset_name = root.split('/')[2]
        if dataset_name not in dataset_names:
            dataset_names.append(dataset_name)
        paths.append(os.path.join(root, name))
```
그 후에 나는 amass_test라고 3D pose를 저장할 새로운 디렉토리를 만들어 줬다.
```
save_root = './amass_test'
save_folders = [folder.replace('./amass_data', './amass_test') for folder in folders]
for folder in save_folders:
    os.makedirs(folder, exist_ok=True)
group_path = [[path for path in paths if name in path] for name in dataset_names]
```
그다음으로 amass to pose에서 root_orient를 각 처음 motion의 orient를 빼주었다.  
빼준 이유는 처음 motion의 orient를 빼주지 않으면 2d projection을 시키게 되면 위에서 보는 듯한 결과가 나오기 때문이다.  
예시로 밑의 gif를 보면 확인할 수 있다.  

<img src="../post_images/amass_default.gif" width="50%"  title="table1" alt=""/>   

만약 root_orient를 처음 motion에 대해서 뒤 모션을 빼주면 다음과 같은 결과를 얻을 수 있다.
<img src="../post_images/amass_motion.gif" width="50%"  title="table1" alt=""/>   

```
ex_fps = 20
def amass_to_pose(src_path, save_path):
    bdata = np.load(src_path, allow_pickle=True)
    fps = 0
    try:
        fps = bdata['mocap_framerate']
        frame_number = bdata['trans'].shape[0]
    except:
#         print(list(bdata.keys()))
        return fps
    
    fId = 0 # frame id of the mocap sequence
    pose_seq = []
    if bdata['gender'] == 'male':
        bm = male_bm
    else:
        bm = female_bm
    down_sample = int(fps / ex_fps)
#     print(frame_number)
#     print(fps)
    root_data=0
    with torch.no_grad():
        for fId in range(0, frame_number, down_sample):
            root_orient = torch.Tensor(bdata['poses'][fId:fId+1, :3]).to(comp_device) # controls the global root orientation
            #tt_test=root_orient[0][2].detach().clone()
            if fId==0:
                root_data=root_orient
            root_orient=root_orient-root_data
            #root_orient[0][1]=tt_test
            pose_body = torch.Tensor(bdata['poses'][fId:fId+1, 3:66]).to(comp_device) # controls the body
            pose_hand = torch.Tensor(bdata['poses'][fId:fId+1, 66:]).to(comp_device) # controls the finger articulation
            betas = torch.Tensor(bdata['betas'][:10][np.newaxis]).to(comp_device) # controls the body shape

            #trans = torch.Tensor(bdata['trans'][fId:fId+1]).to(comp_device)    
            body = bm(pose_body=pose_body, pose_hand=pose_hand, betas=betas, root_orient=root_orient)
            joint_loc = body.Jtr[0] #+ trans
            pose_seq.append(joint_loc.unsqueeze(0))
    pose_seq = torch.cat(pose_seq, dim=0)
    
    pose_seq_np = pose_seq.detach().cpu().numpy()
    np.save(save_path, pose_seq_np)
    return fps
```
root_data에 처음 motion을 저장했다가 다음 frame마다 빼주는 식으로 code를 구성했다.  
그 다음으로 3D Pose를 npy형태로 저장하게 끔 만들었다.
```
import time
for paths in group_path:
    dataset_name = paths[0].split('/')[2]
    pbar = tqdm(paths)
    pbar.set_description('Processing: %s'%dataset_name)
    fps = 0
    for path in pbar:
        #print(path)
        save_path = path.replace('./amass_data', './amass_test')
        save_path = save_path[:-3] + 'npy'
        #print(save_path)
        fps = amass_to_pose(path, save_path)
        
    cur_count += len(paths)
    print('Processed / All (fps %d): %d/%d'% (fps, cur_count, all_count) )
    time.sleep(0.5)
```
### 2D image Projection
이제부터 2D image에 projection을 하는 방법을 설명하고자 한다.  
우선 npy 형식으로 저장한 하나의 data를 우선 얻고 skeleton을 확인한다.  
```
import numpy as np
test=np.load("/home/qazw5741/T2M-GPT/amass_test/BMLhandball/S09_Novice/Trial_upper_right_020_poses.npy")
skeleton = ( (0,1), (1,4), (4,7), (7,10), (0,2), (2,5), (5,8), (8,11), (0,3), (3,6), (6,9), (9,14), (14,17), (17,19), (19, 21), (9,13), (13,16), (16,18), (18,20), (9,12))
```
skeleton은 SMPLH의 regressor 결과로 해당하는 joint는 다음과 같다.
```
     0: 'pelvis',
     1: 'left_hip',
     2: 'right_hip',
     3: 'spine1',
     4: 'left_knee',
     5: 'right_knee',
     6: 'spine2',
     7: 'left_ankle',
     8: 'right_ankle',
     9: 'spine3',
    10: 'left_foot',
    11: 'right_foot',
    12: 'neck',
    13: 'left_collar',
    14: 'right_collar',
    15: 'head',
    16: 'left_shoulder',
    17: 'right_shoulder',
    18: 'left_elbow',
    19: 'right_elbow',
    20: 'left_wrist',
    21: 'right_wrist',
    22: 'left_index1',
    23: 'left_index2',
    24: 'left_index3',
    25: 'left_middle1',
    26: 'left_middle2',
    27: 'left_middle3',
    28: 'left_pinky1',
    29: 'left_pinky2',
    30: 'left_pinky3',
    31: 'left_ring1',
    32: 'left_ring2',
    33: 'left_ring3',
    34: 'left_thumb1',
    35: 'left_thumb2',
    36: 'left_thumb3',
    37: 'right_index1',
    38: 'right_index2',
    39: 'right_index3',
    40: 'right_middle1',
    41: 'right_middle2',
    42: 'right_middle3',
    43: 'right_pinky1',
    44: 'right_pinky2',
    45: 'right_pinky3',
    46: 'right_ring1',
    47: 'right_ring2',
    48: 'right_ring3',
    49: 'right_thumb1',
    50: 'right_thumb2',
    51: 'right_thumb3'
```
우선 3D pose로 plotting 시켜보면 다음 코드를 작성해서 확인 가능하다.  
(나는 보기 편하도록 hand에 대한 pose를 제외한 상태로 진행했다.)
```
import matplotlib.pyplot as plt

for num,frame in enumerate(test):
    fig = plt.figure(figsize=(4,4))
    ax = fig.add_subplot(111, projection='3d')
    ax.set_xlabel("X_axis")
    ax.set_ylabel("Y_axis")
    ax.set_zlabel("Z_axis")
    ax.view_init(90,-90) # default
    #ax.view_init(0,0) # default
    ax.set_xlim([-1, 1])
    ax.set_ylim([0, 2])
    ax.set_zlim([-1, 1])
    skeleton = ( (0,1), (1,4), (4,7), (7,10), (0,2), (2,5), (5,8), (8,11), (0,3), (3,6), (6,9), (9,14), (14,17), (17,19), (19, 21), (9,13), (13,16), (16,18), (18,20), (9,12))
    test_x=[]
    test_y=[]
    test_z=[]
    for i,j in skeleton:
        test_x.append(frame[i][0])
        test_x.append(frame[j][0])
        test_y.append(frame[i][1])
        test_y.append(frame[j][1])
        test_z.append(frame[i][2])
        test_z.append(frame[j][2])
        ax.plot(test_x,test_y,test_z)
        test_x=[]
        test_y=[]
        test_z=[]
    break
```
`set_xlim,set_ylim,set_zlim`은 3d plotting에서 보여줄 범위를 보여준다고 할 수 있고 `view_init`은 보는 각도를 의미한다. (90,-90)을 선택한 이유는 가장 보기 좋기 때문이다.  

<img src="../post_images/3dplot.gif" width="50%"  title="table1" alt=""/>   

3d plot을 하게되면 위의 애니메이션 처럼 나오게 되고 root는 처음 시작할 때 [0,0,0]임을 확인할 수 있다.  
이제 다음으로 2D Projection을 진행할 차례인데 projection에서 가장 중요한건 Camera Intrinsic, Extrinsic parameter이다.  
카메라 외부 파라미터에서 pose에서 z에 해당하는 값에 +4를 더해준 후에 진행을 하였는데 이유는 이러한 작업 없이 바로 projection을 시키게 되면 카메라가 너무 가까워 적절한 pose를 뽑을 수 없기 때문이다.
```
camera_pose = np.eye(4)
camera_pose[:3, 3] = np.array([0, 0, 4])
joint_test=test[0]
res_joint=np.insert(joint_test,3,1,axis=1)
res_joint=res_joint.transpose()
res_joint=np.dot(camera_pose[:3],res_joint)
```
그 후에 대략적인 image shape으로 500,500을 선택했고 이에 따른 focal length estimation 함수를 적용해 focal length를 구했고 princpt는 image의 중앙으로 설정했다.  
`cam2pixel`은 projection 시키는 함수이고 결과에 나오는 z는 버린다고 생각하면 된다.
```
img_shape=[500,500]
princpt=[int(img_shape[0]/2),int(img_shape[1]/2)]
def estimate_focal_length(img_h, img_w):
    return (img_w * img_w + img_h * img_h) ** 0.5
focal_xy=estimate_focal_length(img_shape[0],img_shape[1])
focal=[focal_xy,focal_xy]
def cam2pixel(cam_coord, f, c):
    x = cam_coord[:,0] / cam_coord[:,2] * f[0] + c[0]
    y = cam_coord[:,1] / cam_coord[:,2] * f[1] + c[1]
    z = cam_coord[:,2]
    return np.stack((x,y,z),1)
joint_img_xyz=cam2pixel(res_joint.transpose(),focal,princpt)
```
이렇게 코드를 거치면 projection된다.  
joint_img_xyz를 plotting 시키면 다음과 같다.  
<img src="../post_images/2dprojection_plotting.png" width="50%"  title="table1" alt=""/>   

만약 애니메이션을 보고싶다면 다음 코드를 작성하면 된다.
```
camera_pose = np.eye(4)
camera_pose[:3, 3] = np.array([0, 0, 4])
for num,i in enumerate(test):
    res_joint=np.insert(i,3,1,axis=1)
    res_joint=res_joint.transpose()
    res_joint=np.dot(camera_pose[:3],res_joint)

    img_shape=[500,500]
    princpt=[int(img_shape[0]/2),int(img_shape[1]/2)]
    def estimate_focal_length(img_h, img_w):
        return (img_w * img_w + img_h * img_h) ** 0.5
    focal_xy=estimate_focal_length(img_shape[0],img_shape[1])
    focal=[focal_xy,focal_xy]
    def cam2pixel(cam_coord, f, c):
        x = cam_coord[:,0] / cam_coord[:,2] * f[0] + c[0]
        y = cam_coord[:,1] / cam_coord[:,2] * f[1] + c[1]
        z = cam_coord[:,2]
        return np.stack((x,y,z),1)
    joint_img_h36m_xyz=cam2pixel(res_joint.transpose(),focal,princpt)
    plt.xlim([0, 500])
    plt.ylim([0, 500])
    skeleton = ( (0,1), (1,4), (4,7), (7,10), (0,2), (2,5), (5,8), (8,11), (0,3), (3,6), (6,9), (9,14), (14,17), (17,19), (19, 21), (9,13), (13,16), (16,18), (18,20), (9,12))
    test_x=[]
    test_y=[]
    for i,j in skeleton:
        test_x.append(joint_img_h36m_xyz[i][0])
        test_x.append(joint_img_h36m_xyz[j][0])
        test_y.append(joint_img_h36m_xyz[i][1])
        test_y.append(joint_img_h36m_xyz[j][1])
        plt.plot(test_x,test_y)
        test_x=[]
        test_y=[]
    plt.savefig('./amass_fig/savefig_{}.jpg'.format(str(num).zfill(3)))
    plt.cla()
```

## Next
다음에 진행할 연구는 SMPLH의 vertex정보를 뽑아서 regressor를 돌려 eyes, nose, ear pose를 뽑아볼 계획이다.  
[여기](https://yangchanghee2251.github.io/human/amass_data_projection2/)에 다음 연구가 있다.