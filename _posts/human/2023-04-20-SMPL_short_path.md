---
layout: single
title: "SMPL_mesh short path algorithm (Dijkstra, pygeodesic)"
author_profile: true
categories:
  - human
toc_label: "목차"
toc: true
toc_sticky: true
sidebar:
    nav: "sidebar-category"
---


## Dijkstra Algorithm
내가 하려는 작업은 SMPL Mesh에서 두 Vertex에 대한 shortest path를 구하려는 작업이다. 그냥 두 vertex 사이의 유클리디안 거리를 구하게 되면 3차원이기 때문에 직선 거리를 구하게 되고 mesh에 대한 weight를 계산할 수가 없다. 따라서 shortest path를 이용해 구해보고자 한다.  
다익스트라 알고리즘은 그래프 상에서 가장 가까운 거리를 구하는 방법이다.  
Mesh에 face가 있는데 그 face가 각 메쉬를 연결해주는 연결점이다.(그래프 연결이라고 생각하면 편하다.) 이를 이용해 graph shortest path를 구해보고자 한다.

### requirements
내가사용한 python package이다. vtk, Pillow는 사용하지 않아도 문제가 없어서 굳이 받을 필요는 없다.
```
oldest-supported-numpy
matplotlib
pygeodesic==0.1.8
vtk
notebook==6.0.3
Pillow
opencv-python
```

### 다익스트라 알고리즘 구현
준비물은 SMPL을 거쳐서 나온 Vertex와 faces가 필요하다.  
(나는 SMPLH에서 나온 Vertex와 SMPLH faces를 사용했다. SMPL에서 나온 vertex, faces를 사용해도 무관하다.)  
우선 point 정보가 담겨있는 vertex와 face를 load한다.
```
import numpy as np
from PIL import Image
faces=np.load('smplh_faces.npy')
vertxs=np.load('Trial_upper_right_020_poses.vertexnpy.npy')
```
이렇게 load한 vertex중 한 사람에 대한 vertex만 사용하고 이제 각 연결된 graph를 graph_dict에 저장한다.
```
vertex=vertxs[0]
graph_dict={}
for face in faces:
    for num,graph in enumerate(face):
        if not graph in graph_dict:
            graph_dict[graph]=[]
        for num1,check in enumerate(face):
            if num==num1:
                continue
            if check in graph_dict[graph]:
                continue
            else:
                graph_dict[graph]+=[check]
```
이렇게 저장하면 graph_dict은 다음과 같은 output을 보인다.
```
{1: [2, 0, 4, 5, 235, 133],
 2: [1, 0, 3, 4, 6, 188],
 0: [1, 2, 3, 133, 132, 134],
 3: [0, 2, 6, 7, 134, 135],
 4: [2, 1, 5, 105, 188, 189],
 5: [4, 1, 104, 105, 234, 235],
 6: [2, 3, 7, 192, 188, 191],
 7: [3, 6, 135, 136, 192, 193],
 9: [10, 8, 157, 245, 265, 261],
 10: [9, 8, 11, 156, 155, 157],
 8: [9, 10, 11, 113, 261, 262],
 11: [8, 10, 156, 246, 110, 113],
 13: [14, 12, 244, 137, 236, 264, 239],
 14: [13, 12, 15, 137, 138, 139],
 12: [13, 14, 15, 265, 264],
 15: [12, 14, 265, 266, 154, 139],
 17: [18, 16, 25, 24, 41, 238],
 18: [17, 16, 19, 341, 237, 238],
 16: [17, 18, 19, 24, 26, 27],
 19: [16, 18, 340, 227, 341, 26],
 21: [22, 20, 227, 349, 353, 354],
 22: [21, 20, 23, 349, 407, 348],
 20: [21, 22, 23, 36, 26, 227, 27],
 23: [20, 22, 36, 37, 328, 407],
 25: [17, 24, 44, 276, 42, 95, 41],
...
 997: [996, 995, 998, 1040, 1000, 1038],
 995: [996, 997, 998, 1061, 1522, 1523],
 998: [995, 997, 1061, 1062, 999, 1000],
 1000: [1001, 999, 1036, 1038, 998, 997],
 ...}
```
이렇게 저장한 graph_dict에서 각 vertex들의 거리를 구해 다른 dictionary에 저장한다.
```
distance_graph_dict={}
for graph in graph_dict.keys():
    distance_graph_dict[graph]={}
    for vert_id in graph_dict[graph]:
        distance_graph_dict[graph][vert_id]=np.sqrt(sum((vertex[graph]-vertex[vert_id])**2))
```
이렇게 저장했다면 이제 다익스트라 알고리즘을 구동시키면 된다. 다익스트라 알고리즘은 다음과 같은 code를 사용했으며 나는 path도 보고싶어 stack이라는 dictionary를 만들어 return 시켰다.
```
import heapq 

def dijkstra(graph, start):
  distances = {node: float('inf') for node in graph}
  distances[start] = 0 
  stack = {node: [] for node in graph}
  queue = []
  heapq.heappush(queue, [distances[start], start])

  while queue:
    current_distance, current_destination = heapq.heappop(queue)
    #print(current_distance,current_destination)
    if distances[current_destination] < current_distance:
      continue
    
    for new_destination, new_distance in graph[current_destination].items():
      distance = current_distance + new_distance 
      if distance < distances[new_destination]: 
        distances[new_destination] = distance
        stack[new_destination]+=[current_destination]
        heapq.heappush(queue, [distance, new_destination]) 
    
  return distances,stack
```
다익스트라 알고리즘을 구현했으면 graph와 start input이 있는데 graph input은 위에서 만들었던 distance_graph_dict를 넣으면 되고 start는 source destination(출발점)을 넣으면 된다.  
나는 시작점을 5000으로 설정했다.
```
test,graph=dijkstra(distance_graph_dict,5000)
```
이렇게 하면 test => distances, graph => stack으로 구성이 되는데 distance는 5000번 vertex와 다른 6890개의 vertex의 최소 거리를 나타내고 graph는 최소거리를 가기 위한 vertex index들이 들어있다.  
만약 5000번 vertex와 300번 vertex의 거리를 보고 싶으면 아래와 같이 코드를 치면되고
```
test[300]
```
값은 1.3702188509147473가 나오게 된다.  
graph의 형태는 다음과 같이 되어있다.
```
{1: [4],
 2: [6],
 0: [2, 3],
 3: [7],
 4: [189, 105],
 5: [105, 104],
 6: [192],
 7: [193],
 9: [245],
 10: [9],
 8: [9],
 11: [8],
 13: [137],
 14: [138, 139],
 12: [14, 15],
 15: [139, 154],
 17: [41],
 18: [238, 17],
 16: [17],
 19: [18, 16],
 21: [20],
 22: [20],
 20: [27],
 23: [36],
 25: [95],
...
 997: [1040],
 995: [997],
 998: [997],
 1000: [1038, 997],
 ...}
```
이는 각 vertex를 가기 위해서 거쳐야 하는 vertex를 의미하고 path를 얻고 싶다면 아래의 코드를 구현하면 된다.
```
test=[300]
output=[300]
while test:
    t1=graph[test.pop(0)]
    output+=t1
    test=t1
```
test.pop(0)를 통해 거쳐간 vertex를 output에 추가적으로 넣어주면 지금까지 거쳐간 vertex를 확인할 수 있다.  
output의 형태
```
[300, 257, 424, 829, 453, 828, 3470, ... 4995, 4998, 4997, 4999, 5000]
```
이제 이걸 reverse sort를 시킨다.
```
output=sorted(output,reverse=True)
vt_test=[]
for i in output:
    vt_test.append(vertex[i])
    vertex=np.delete(vertex,i,0)
print(vertex.shape)
```
그다음 path에 해당하는 vertex들을 빼준 후에 vt_test에 넣어준다.
```
for angle in range(0, 360):
    fig = plt.figure(figsize=(9,9))
    ax = fig.add_subplot(111, projection='3d')
    ax.set_xlabel("X_axis")
    ax.set_ylabel("Y_axis")
    ax.set_zlabel("Z_axis")
    #ax.view_init(90,-90) # default
    #ax.view_init(0,0) # default
    ax.view_init(angle, -90)
    ax.set_xlim([-1.5, 1.5])
    ax.set_ylim([-1.5, 1.5])
    ax.set_zlim([-1.5, 1.5])
    test_x=vertex[:,0]
    test_y=vertex[:,1]
    test_z=vertex[:,2]
    #ax.view_init(0,0) # default
    ax.scatter(test_x,test_y,test_z,s=0.1,c="gray")
    test_x=vt_test[:,0]
    test_y=vt_test[:,1]
    test_z=vt_test[:,2]
    ax.scatter(test_x,test_y,test_z,s=2,c="blue")
    plt.savefig('./amass_fig/savefig_{}.jpg'.format(str(angle).zfill(3)))
    plt.close(fig)
```
마지막으로 360도 회전을 통해 vertex를 확인하는 코드를 짯다. 결과는 다음과 같다.

<img src="../post_images/dijkstra_algorithm2.gif" width="100%"  title="table1" alt=""/>   

결과를 확인해보면 graph 상에서 움직이는 shortest path다보니 중간중간에 선이 좌우좌우로 가는 것을 확인할 수 있다.  
따라서 이 문제를 해결하기 위해서 geodesic distance를 이용해 해결했다.

### pygeodesic
웹 서핑을 계속하다보니 pygeodesic이라는 기막힌 package가 있어서 이를 활용했다.  
[pygeodesic](https://github.com/mhogg/pygeodesic)를 확인하면 다양한 example을 확인할 수 있다.  
우선 결과부터 보자면

<img src="../post_images/pygeodesic_algorithm.gif" width="100%"  title="table1" alt=""/>   

이전의 다익스트라 알고리즘의 문제를 해결하는 결과를 보여주고 있다. 굉장히 매끈한 선을 보여주고 있으며 mesh위를 지나가는 선이 만들어진 것을 확인할 수 있다.

우선 위 pygeodesic에 들어가게 되면 github 레포토지를 받아서 `python setup.py install`를 해야한다고 한다. 근데 내가 직접 해본결과 굳이 안해도 되는 것으로 확인 됫다.  

Import pygeodesic
```
# Imports
import pygeodesic
import pygeodesic.geodesic as geodesic
import numpy as np
```
우선 import를 진행하고 version을 확인해보면
```
pygeodesic.__version__
'0.1.8'
```
가 나오게 된다.  
이렇게 아까와 같이 SMPL vertex, face를 load해오자
```
import numpy as np
faces=np.load('smplh_faces.npy')
vertxs=np.load('Trial_upper_right_020_poses.vertexnpy.npy')
vertex=vertxs[0]
```
이 pygeodesic의 장점은 vertex와 face를 그냥 넣어주면 끝이라는 것이다.
```
geoalg = geodesic.PyGeodesicAlgorithmExact(vertex, faces)
```
아마도 내장 함수에서 거리를 자동으로 계산해주는 것으로 판단된다.  
geodesic distance는 일반적인 그래프 알고리즘과 다르게 삼각형이라는 polygon 특성을 이용해서 풀어내는 문제인데 이를 자세히 설명한 유튜브가 있는데 [여기](https://www.youtube.com/watch?v=DbNEsryLULE)를 한번 보는걸 추천한다.  
또한 이 패키지의 장점은 `시작점, 도착점`을 그냥 정해주면 된다는 것인데
```
sourceIndex = 5000
targetIndex = 300
distance, path = geoalg.geodesicDistance(sourceIndex, targetIndex)
```
3줄이면 끝난다! 심지어 path까지 찾아서 보내준다.  
아까와 같이 5000 => 300을 지정하여 distance를 확인해보니
1.3702188509147473 => 1.3089782387619278 아까보다 좀 더 길이가 줄어든 것을 확인할 수 있었다.  
이제 이를 3d plotting 시켜서 시각화 해보면 아까의 결과를 확인할 수 있다.
```
import matplotlib.pyplot as plt
for angle in range(0, 360):
    fig = plt.figure(figsize=(9,9))
    ax = fig.add_subplot(111, projection='3d')
    ax.set_xlabel("X_axis")
    ax.set_ylabel("Y_axis")
    ax.set_zlabel("Z_axis")
    ax.view_init(90,-90) # default
    #ax.view_init(0,0) # default
    ax.view_init(angle, -90)
    ax.set_xlim([-1.5, 1.5])
    ax.set_ylim([-1.5, 1.5])
    ax.set_zlim([-1.5, 1.5])
    test_x=vertex[:,0]
    test_y=vertex[:,1]
    test_z=vertex[:,2]
    ax.scatter(test_x,test_y,test_z,s=0.1,c="gray")
    test_x=path[:,0]
    test_y=path[:,1]
    test_z=path[:,2]
    ax.scatter(test_x,test_y,test_z,s=2,c="blue")
    plt.savefig('./amass_fig/savefig_{}.jpg'.format(str(angle).zfill(3)))
    plt.close(fig)
```

### 후기
동적 프로토콜 과제를 위해서 진행해봤는데 생각보다 알고리즘에 대한 개념이 많이 필요한 것 같다. 다익스트라에 업그레이드 버젼인 A * 알고리즘도 있는데 이는 장애물이 있을 때 좀 더 빠른 결과를 보여주는 알고리즘이며 만약 없을 시 다익스트라 알고리즘과 비슷한 속도를 보인다고 한다. mesh의 경우에는 장애물이 없기 때문에 그냥 다익스트라 알고리즘과 속도차이가 거의 없을 것으로 판단해 그냥 하나만 진행해 보았다.