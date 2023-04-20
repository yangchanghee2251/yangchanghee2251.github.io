---
layout: single
title: "Amass dataset 2D Projection (2)"
author_profile: true
categories:
  - human
toc_label: "목차"
toc: true
toc_sticky: true
sidebar:
    nav: "sidebar-category"
---


## 이전 연구
[여기](https://yangchanghee2251.github.io/human/amass_data_projection/)에 들어가면 이전에 했던 연구를 볼 수 있다.  
이번에 해볼 연구는 vertex를 뽑아서 Regressor를 거친 후에 pose를 뽑아 projection 시키는 것이다.

### requirements
이번에 진행할 연구는 SMPLH에 smpl joint regressor를 사용하면 적절한 position이 나오는지와 projection이 제대로 되는지 확인하기 위한 연구이다.
```
pip install opencv-python
pip install torchgeometry
pip install transforms3d
pip install pycocotools
pip install git+https://github.com/scottandrews/chumpy.git@fe51783e0364bf1e9b705541e7d77f894dd2b1ac
pip install pyrender
pip install trimesh
pip install human-body-prior
pip install easydict
pip install tqdm
```
이번 requirements가 바뀐 이유는 SMPL에 있는 J_regressor를 사용하기 위함이며 [3DCrowdNet](https://github.com/hongsukchoi/3DCrowdNet_RELEASE)에 있는 github code를 참고했다.  
좀더 자세히 내가 하려는 연구를 설명하면, Amass Dataset에 있는 data를 J_regressor를 통해 내가 원하는 30개의 data를 얻어 2D image에 Projection을 시키는게 나의 주된 목적이다.  
이전에 했던 SMPLH의 3D pose는 다양한 pose dataset에서 존재하는 joint index를 전부 커버할 수 없기 때문에 SMPL model에 있는 J_regressor로 새롭게 만들었으며 각 Joint는 다음과 같다.
```
joints_name = ('Pelvis', 'L_Hip', 'R_Hip', 'Torso', 'L_Knee', 'R_Knee', 'Spine', 'L_Ankle', 'R_Ankle', 'Chest', 'L_Toe', 'R_Toe', 'Neck', 'L_Thorax', 'R_Thorax',
                            'Head', 'L_Shoulder', 'R_Shoulder', 'L_Elbow', 'R_Elbow', 'L_Wrist', 'R_Wrist', 'L_Hand', 'R_Hand', 'Nose', 'L_Eye', 'R_Eye', 'L_Ear', 'R_Ear', 'Head_top')
skeleton = ( (0,1), (1,4), (4,7), (7,10), (0,2), (2,5), (5,8), (8,11), (0,3), (3,6), (6,9), (9,14), (14,17), (17,19), (19, 21), (21,23), (9,13), (13,16), (16,18), (18,20), (20,22), (9,12), (12,24), (24,15), (24,25), (24,26), (25,27), (26,28), (24,29) )
```
결론부터 말해보자면 SMPL, SMPLH의 vertex 숫자는 6890개로 동일하여 regressor가 적용될 것으로 판단했으면 내생각대로 잘 되는걸 확인했다.  

<img src="../post_images/amass_to_smpl.gif" width="25%"  title="table1" alt=""/>   

위 애니메이션은 3D plotting해서 SMPL regressor가 과연 SMPLH에서 나온 vertex를 잘 regressor할 것인지 궁금하여 plotting을 시킨 것이고 밑에 애니메이션은 이를 projection 했을 때 잘 되는 지 확인을 하였다.
<img src="../post_images/amass_projection.gif" width="100%"  title="table1" alt=""/>   

두 영상 모두 잘 되는 것을 확인하였고 이제부터 어떻게 만드는지 알려주고한다.

### SMPL Model regressor
```
import numpy as np
test=np.load("/home/qazw5741/T2M-GPT/amass_test/BMLhandball/S09_Novice/Trial_upper_right_020_poses.npy")
vertex=np.load("/home/qazw5741/T2M-GPT/amass_test/BMLhandball/S09_Novice/Trial_upper_right_020_poses.vertexnpy.npy")
skeleton = ( (0,1), (1,4), (4,7), (7,10), (0,2), (2,5), (5,8), (8,11), (0,3), (3,6), (6,9), (9,14), (14,17), (17,19), (19, 21), (9,13), (13,16), (16,18), (18,20), (9,12))
```
우선 우리가 이전에 저장한 pose와 vertex를 불러와야 한다. 내가 이전 포스팅에는 vertex를 저장하는 방법에 대해서 소개하지 않아서 간단히 소개하고 넘어가겠다.
[human_body_model](https://github.com/nghorbani/human_body_prior/blob/4c246d8a83ce16d3cff9c79dcf04d81fa440a6bc/src/human_body_prior/body_model/body_model.py#L267)여기를 보면
```
        verts, Jtr = lbs(betas=shape_components, pose=full_pose, v_template=v_template,
                            shapedirs=shapedirs, posedirs=self.posedirs,
                            J_regressor=self.J_regressor, parents=self.kintree_table[0].long(),
                            lbs_weights=self.weights, joints=joints, v_shaped=v_shaped,
                            dtype=self.dtype)
        Jtr = Jtr + trans.unsqueeze(dim=1)
        verts = verts + trans.unsqueeze(dim=1)
        res = {}
        res['v'] = verts
        res['f'] = self.f
        res['Jtr'] = Jtr  # Todo: ik can be made with vposer
        # res['bStree_table'] = self.kintree_table
```
이렇게 코드가 구성되어있음을 볼 수 있는데 lbs를 통해 vertex, j_regressor를 통해 나온 3D pose가 나옴을 알 수 있고 이전에 Jtr을 호출해 3D pose를 가져왔던 기억이 날 것이다. 나는 여기서 v를 호출해 vertex를 가져왔다.
```
ex_fps = 20
def amass_to_pose(src_path, save_path,save_path1):
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
    vertex_seq = []
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
            vertex=body.v[0]
            pose_seq.append(joint_loc.unsqueeze(0))
            vertex_seq.append(vertex.unsqueeze(0))
    pose_seq = torch.cat(pose_seq, dim=0)
    vertex_seq = torch.cat(vertex_seq, dim=0).detach().cpu().numpy()
    
    pose_seq_np = pose_seq.detach().cpu().numpy()
    np.save(save_path, pose_seq_np)
    np.save(save_path1,vertex_seq)
    return fps
```
이렇게 vertex_seq라는 list를 만들어준다음 저장을 해주었다.  
이제 vertex가 과연 잘 저장이 되었나 확인을 하기 위해서는 다음과 같은 작업을 통해 확인할 수 있다.
```
vertex[0].shape
test_x=vertex[0,:,0]
test_y=vertex[0,:,1]
test_z=vertex[0,:,2]
fig = plt.figure(figsize=(4,4))
ax = fig.add_subplot(111, projection='3d')
ax.set_xlabel("X_axis")
ax.set_ylabel("Y_axis")
ax.set_zlabel("Z_axis")
ax.view_init(90,-90) # default
#ax.view_init(0,0) # default
ax.set_xlim([-1.5, 1.5])
ax.set_ylim([-1.5, 1.5])
ax.set_zlim([-1.5, 1.5])
ax.scatter(test_x,test_y,test_z,s=2)
```
위 코드를 확인하면 아래와 같은 이미지가 나올 것이다.
여기까지 왔다면 이제 바로 Projection을 진행해보자!  

<img src="../post_images/vertex_amass.png" width="100%"  title="table1" alt=""/>   

### 3DCrowdNet Code
나같은 경우에는 3DCrowdNet에 있는 Code를 사용하여 진행했으며 다른 SMPL model을 사용해서 j_regressor를 뽑아서 무관하다.  
내 path 위치는 `/home/qazw5741/3DCrowdNet/main/test.ipynb`에서 작업을 했다.
```
import sys
sys.path.append("/home/qazw5741/3DCrowdNet/")
from common.utils.smpl import SMPL
smpl = SMPL()
face = smpl.face
joint_regressor = smpl.joint_regressor
```
우선 SMPL에 존재하는 J_regressor를 뽑아준 후에 3D에 plotting 시켜봤다.
```
import matplotlib.pyplot as plt
for n_v,vert in enumerate(vertex):
    h36m_from_smpl = np.dot(joint_regressor, vert)
    fig = plt.figure(figsize=(12,12))
    ax = fig.add_subplot(111, projection='3d')
    ax.set_xlabel("X_axis")
    ax.set_ylabel("Y_axis")
    ax.set_zlabel("Z_axis")
    ax.view_init(90,-90) # default
    #ax.view_init(0,0) # default
    ax.set_xlim([-1.5, 1.5])
    ax.set_ylim([-1.5, 1.5])
    ax.set_zlim([-1.5, 1.5])
    skeleton = ( (0,1), (1,4), (4,7), (7,10), (0,2), (2,5), (5,8), (8,11), (0,3), (3,6), (6,9), (9,14), (14,17), (17,19), (19, 21), (21,23), (9,13), (13,16), (16,18), (18,20), (20,22), (9,12), (12,24), (24,15), (24,25), (24,26), (25,27), (26,28), (24,29) )
    test_x=vertex[n_v,:,0]
    test_y=vertex[n_v,:,1]
    test_z=vertex[n_v,:,2]
    #ax.view_init(0,0) # default
    ax.scatter(test_x,test_y,test_z,s=0.1,c="gray")
    test_x=[]
    test_y=[]
    test_z=[]
    for i,j in skeleton:
        test_x.append(h36m_from_smpl[i][0])
        test_x.append(h36m_from_smpl[j][0])
        test_y.append(h36m_from_smpl[i][1])
        test_y.append(h36m_from_smpl[j][1])
        test_z.append(h36m_from_smpl[i][2])
        test_z.append(h36m_from_smpl[j][2])
        ax.plot(test_x,test_y,test_z,linewidth="4")
        test_x=[]
        test_y=[]
        test_z=[]
    break
```

<img src="../post_images/j_regressor_amass.png" width="100%"  title="table1" alt=""/>   

3D plotting은 제대로 되는걸 확인 했으니 projection이 적절히 되는지 확인해 보았다.
```
for n_v,vert in enumerate(vertex):
    h36m_from_smpl = np.dot(joint_regressor, vert)
    camera_pose = np.eye(4)
    camera_pose[:3, 3] = np.array([0, 0, 4])
    res_joint=np.insert(h36m_from_smpl,3,1,axis=1)
    res_joint=res_joint.transpose()
    res_joint=np.dot(camera_pose[:3],res_joint)
    vertex_test=vertex[n_v]
    res_vert=np.insert(vertex_test,3,1,axis=1)
    res_vert=res_vert.transpose()
    res_vert=np.dot(camera_pose[:3],res_vert)
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
    joint_img_xyz1=cam2pixel(res_joint.transpose(),focal,princpt)
    joint_img_xyz1_vert=cam2pixel(res_vert.transpose(),focal,princpt)
    plt.xlim([0, 500])
    plt.ylim([0, 500])
    skeleton = ( (0,1), (1,4), (4,7), (7,10), (0,2), (2,5), (5,8), (8,11), (0,3), (3,6), (6,9), (9,14), (14,17), (17,19), (19, 21), (21,23), (9,13), (13,16), (16,18), (18,20), (20,22), (9,12), (12,24), (24,15), (24,25), (24,26), (25,27), (26,28), (24,29) )
    test_x=[]
    test_y=[]
    test_x1=joint_img_xyz1_vert[:,0]
    test_y1=joint_img_xyz1_vert[:,1]
    for i,j in skeleton:
        test_x.append(joint_img_xyz1[i][0])
        test_x.append(joint_img_xyz1[j][0])
        test_y.append(joint_img_xyz1[i][1])
        test_y.append(joint_img_xyz1[j][1])
        plt.plot(test_x,test_y)
        test_x=[]
        test_y=[]
    plt.scatter(test_x1,test_y1,s=0.1)
    plt.savefig('./amass_fig/savefig_{}.jpg'.format(str(n_v).zfill(3)))
    #plt.cla()
    break
```

<img src="../post_images/projection_j_regressor.png" width="100%"  title="table1" alt=""/>   

## 후기
생각보다 간단했으며 다음에 진행해볼 실험은 SMPL parameter를 SMPLH, SMPL Model에 넣었을 때 나오는 Vertex가 다른가? 라는 의문으로 시작되 확인해볼려고 한다.