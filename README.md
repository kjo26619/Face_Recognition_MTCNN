# Face_Recognition_MTCNN
Face_Recognition_MTCNN

# MTCNN

MTCNN은 논문 Joint Face Detection and Alignment using Multi-task Cascaded Convolutional Networks 에서 나온 모델로

얼굴 인식을 할 때 Accuracy를 높이기 위해 3개의 네트워크를 사용하는 모델입니다.

![img1](https://github.com/kjo26619/Face_Recognition_MTCNN/blob/main/MTCNN.PNG)
< 출처 : https://arxiv.org/abs/1604.02878 >

기본적인 Convolution Layer와 총 3개의 네트워크 P-Net, R-Net, O-Net으로 이루어져 있습니다.

그리고 각 네트워크마다 총 3개의 결과가 나오는데 다음과 같습니다.

1. Face Classification : 얼굴인지 확인 여부가 나옵니다.

2. Bounding Box Regression : 얼굴이 위치한 곳을 네모난 박스(x, y, width, height)로 출력합니다.

3. Facial Landmark Localizaiton : 얼굴의 주요 랜드마크(눈, 코, 입 등)의 위치를 나타냅니다.

각각 네트워크는 다른 크기의 이미지를 학습하여 결과를 보여주고 학습한 결과를 다음 네트워크로 옮겨줍니다. (P-Net -> R-Net -> O-Net)

## P-Net (Proposal Network)

P-Net은 12x12x3의 이미지가 들어옵니다. 매우 작은 크기이며 작은 사이즈여도 얼굴을 찾기 위해 이러한 작업을 합니다.

P-Net을 통해서 나온 

## R-Net (Refine Network)

## O-Net (Output Network)
