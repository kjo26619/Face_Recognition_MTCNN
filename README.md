# Face_Recognition_MTCNN
Face_Recognition_MTCNN

# MTCNN

MTCNN은 논문 Joint Face Detection and Alignment using Multi-task Cascaded Convolutional Networks 에서 나온 모델로

얼굴 인식을 할 때 Accuracy를 높이기 위해 3개의 네트워크를 사용하는 Cascade 모델입니다.

![img1](https://github.com/kjo26619/Face_Recognition_MTCNN/blob/main/MTCNN.PNG)
< 출처 : https://arxiv.org/abs/1604.02878 >

기본적인 Convolution Layer와 총 3개의 네트워크 P-Net, R-Net, O-Net으로 이루어져 있습니다.

그리고 각 네트워크마다 총 3개의 결과가 나오는데 다음과 같습니다.

1. Face Classification : 얼굴인지 확인 여부가 나옵니다.

2. Bounding Box Regression : 얼굴이 위치한 곳을 네모난 박스(x, y, width, height)로 출력합니다.

3. Facial Landmark Localizaiton : 얼굴의 주요 랜드마크(눈, 코, 입 등)의 위치를 나타냅니다.

각각 네트워크는 다른 크기의 이미지를 학습하여 결과를 보여주고 학습한 결과를 다음 네트워크로 옮겨줍니다. (P-Net -> R-Net -> O-Net)

## P-Net (Proposal Network)

P-Net을 하기 전 이미지를 다양하게 줄이는 작업을 합니다. 이는 작은 사이즈여도 얼굴을 찾기 위해 이러한 작업을 합니다.

P-Net은 12x12x3의 이미지가 들어옵니다. 이미지를 돌면서 얼굴을 검출합니다.

P-Net의 Output에서는 Bounding Box Regression Vector와 Facial Windows 후보군이 나옵니다.

그리고 Bounding Box Regression Vector를 기반으로 후보군을 보정합니다.

P-Net에서는 NMS(Non-Maximum Suppression)를 통해서 Bounding Box 후보군을 줄입니다.

### NMS (Non-Maximum Supperession)

NMS는 얼굴이나 오브젝트 검출 결과에서 후보군을 줄여 연산량을 줄이는 효과를 가져옵니다.

NMS를 위해서는 위해서는 먼저 IoU(Intersection over Union)를 계산해야 합니다.

IoU는 Jaccard Index라고도 불리는데, 전체 Union 박스에서 겹치는 부분을 나타내는 지표입니다. 겹치는 부분이 많을 수록 값이 높아집니다.

결국 NMS는 가장 높은 IoU를 가진 박스를 제외하고 모두 제거합니다.

### Bounding Box Regression

찾아낸 Bouding Box가 얼굴을 포함은 하나 정확하게 감싸지 않을 수 있습니다.

이러한 Bounding Box는 학습을 하는데 안좋게 작용할 수 있습니다. 그렇기에 얼굴을 중앙에 두고 전체적으로 감쌀 수 있도록 조정하는 작업이 Bounding Box Regression입니다.

Bounding Box Regression은 MTCNN 결과로 나온 Bounding Box P와 실제 값(Ground Truth) Bounding Box G와 비교를 해서 조정을 합니다.

Bounding Box는 x, y, width, height로 결과가 나옵니다. 즉, ![eq1](https://latex.codecogs.com/gif.latex?P%5Ei%3D%28P_x%5Ei%2C%20P_y%5Ei%2C%20P_w%5Ei%2C%20P_h%5Ei%29%20%5C%20%5C%20%5C%20G%5Ei%3D%28G_x%5Ei%2C%20G_y%5Ei%2C%20G_w%5Ei%2C%20G_h%5Ei%29) 로 나옵니다.

Bounding Box Regression을 수식으로 나타내면 다음과 같습니다.

![eq2](https://latex.codecogs.com/gif.latex?t_x%20%3D%20%28P_x-P_x%5E*%29/P^*_w%20%5C%20%5C%20%5C%20%5C%20%281%29)

![eq3](https://latex.codecogs.com/gif.latex?t_y%20%3D%20%28P_y-P_y%5E*%29/P^*_h%20%5C%20%5C%20%5C%20%5C%20%282%29)

![eq4](https://latex.codecogs.com/gif.latex?t_w%20%3D%20log%28P_w/P%5E*_w%29%20%5C%20%5C%20%5C%20%5C%20%283%29)

![eq5](https://latex.codecogs.com/gif.latex?t_h%20%3D%20log%28P_h/P%5E*_h%29%20%5C%20%5C%20%5C%20%5C%20%284%29)

![eq6](https://latex.codecogs.com/gif.latex?%5Chat%7Bt%7D_x%20%3D%20%28G_x-P%5E*_x%29/P_w%5E*%20%5C%20%5C%20%5C%20%5C%20%285%29)

![eq7](https://latex.codecogs.com/gif.latex?%5Chat%7Bt%7D_y%20%3D%20%28G_y-P%5E*_y%29/P_h%5E*%20%5C%20%5C%20%5C%20%5C%20%286%29)

![eq8](https://latex.codecogs.com/gif.latex?%5Chat%7Bt%7D_w%20%3D%20log%28G_w/P_w%5E*%29%20%5C%20%5C%20%5C%20%5C%20%287%29)

![eq9](https://latex.codecogs.com/gif.latex?%5Chat%7Bt%7D_h%20%3D%20log%28G_h/P_h%5E*%29%20%5C%20%5C%20%5C%20%5C%20%288%29)

여기서 ![pstar](https://latex.codecogs.com/gif.latex?P%5E*)는 조정한 Bounding Box입니다. 

Bouding Box를 조정해 실제 박스와 거의 유사하게 이동시킵니다.

## R-Net (Refine Network)

R-Net은 P-Net을 통해서 Bounding Box Regression Vector 리스트를 얻은 후 Refine 하는 네트워크입니다.

P-Net에서 구해낸 모든 리스트를 24x24x3으로 조정합니다. 그리고 R-Net을 통과하여 더욱 정교하게 Bounding Box Regression Vector를 찾아냅니다.

역시 마찬가지로, NMS를 통해 후보군을 줄입니다.

## O-Net (Output Network)

O-Net은 R-Net을 통해서 얻어낸 Bounding Box Regression Vector 리스트를 모두 48x48로 조정한 뒤 네트워크를 통과합니다.

R-Net과 거의 유사하지만, 더욱 자세하게 얼굴 영역을 식별하기 위해 Landmark를 검출합니다.

# MTCNN - CNN Architectures

MTCNN 논문에서는 얼굴 인식에 대한 2가지 CNN의 한계점을 설명합니다.

1. Convolution Layer의 일부 필터는 다양하게 식별하는 능력을 제한합니다.

2. 다른 다중 클래스 오브젝트 감지와 분류와 비교하여, 얼굴 감지는 이진 분류 작업이므로 레이어 당 필터 수가 더 적을 수 있습니다.

그래서 필터 수를 줄이고 5x5 필터를 3x3 필터로 변경하여 연산을 줄입니다. 그리고 레이어의 깊이를 높여서 성능이 좋게 만듭니다.

그리고 PReLU를 적용하여 단순한 비선형 함수가 아닌 학습이 가능한 함수를 채택합니다.

# MTCNN - Loss Function

MTCNN에서 Loss Function은 네트워크 출력마다 다른 Loss를 택하고 가중치를 두어 합친 뒤 줄이는 방향으로 학습합니다.

## Classification Loss Function

먼저 Classification의 Loss Function은 다음과 같습니다.

![loss1](https://latex.codecogs.com/gif.latex?L%5E%7Bdet%7D_i%3D%20-%28y%5E%7Bdet%7D_i%20%5C%20log%28p_i%29%20&plus;%20%281-%20y%5E%7Bdet%7D_i%29%281-log%28p_i%29%29%29)

Log Loss 형태를 가지며 Cross-Entropy Loss를 채택했습니다.

여기서, ![pi](https://latex.codecogs.com/gif.latex?p_i)는 샘플 ![pi](https://latex.codecogs.com/gif.latex?x_i)이 얼굴일 확률입니다.  ![yi](https://latex.codecogs.com/gif.latex?y^{det}_i) 는 0~1의 Ground-truth Label입니다.

## Bounding Box Regression Loss Function

다음 Bounding Box Regression Loss Function은 Euclidean Loss Function을 채택했습니다.

![loss2](https://latex.codecogs.com/gif.latex?L%5E%7Bbox%7D_i%3D%5Cleft%20%5C%7C%20%5Chat%7By%7D%5E%7Bbox%7D_i%20-%20y_i%5E%7Bbox%7D%20%5Cright%20%5C%7C%5E2_2)

