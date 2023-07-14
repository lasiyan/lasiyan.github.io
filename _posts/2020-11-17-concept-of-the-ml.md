---
title: 머신 러닝 기본 개요
date: 2023-07-12 16:07:00 +0900
categories: [Stack, A.I]
tag: [Machine Learning, 머신러닝, AI]
---

## 학습 방법론

### Supervised Learning
- 지도학습은 학습에 필요한 데이터에 클래스 라벨이 이미 붙어 있는 경우이다.
- 클래스 라벨은 어떤 인스턴스(ex. 사람, 도시, 차, 음식 등)의 데이터가 주어졌을 경우 그 인스턴스가 어느 분류에 속하는지 나타낸다.
- 클래스는 연속된 값(continuous value) 또는 분리되어 나누어지는 값(discrete)로 분류된다.
- input feature(데이터의 클래스)를 직접 입력해 주어야 하는 한계가 있다.

예시
- 인스턴스. 인스턴스: 소리
- 데이터: 강아지 소리, 아기 울음 소리, 여성 목소리, 자동차 소리, …
- 클래스: 강아지 소리 클래스, 아기울음 의미 클래스, 자동차 소리 구별 클래스, 등

### Unsupervised Learning
- 위와 같은 클래스가 붙지 않은 경우 비지도 학습으로 분류된다.
- 분류가 되지 않은 데이터에서 특징을 자동으로 학습하는 알고리즘
- 최상의 컨디션 경우를 생각해보면 지도학습이 비지도 학습보다 높은 예측률보인다.

---

## 과정

### Data Acquisition
- 데이터를 수집한다.
- 이렇게 수집된 결과물을 Raw Data라 한다.
- ex. 카메라로 고객의 얼굴 촬영, 마이크로 사람 목소리 녹음

### Feature extraction
- 기계가 인식하기 쉽도록 수집된 데이터에서 특징 추출
- ex. 목소리의 톤, 대역폭, 라벨 분류 or 사진에서 왼쪽 눈의 좌표값 등

> ### Feature extraction Detail
- Feature에 해당하는 정보를 수집하고 선정하는 것은 Machine Learning에서 매우 중요
- 어떤 Feature를 사용하느냐에 따라 기계 학습의 성능 결정
- 너무 많은 Feature는 기계의 연산량에 한계를 가져오고
- 혹, 목표와 상관없는 Feature는 잘못된 예측을 불러온다
- 따라서 적절한 Feature 선정을 통해 효율적인 기계학습을 구현하는 것이 엔지니어의 목표

### Classifier
- 해당 feature를 기존에 저장되어 있던 data와 비교하여 가장 유사도가 높은 ID를 찾아낸다.
- 이렇게 기존 data와 Feature에 대한 매칭 정보 집합을 Classifier라 하고,  
Feature를 통해 더 많은 Classifier를 만드는,  이러한 과정을 Training이라 한다.
- training 된 데이터 집합인 classifier에 feature를 입력해 넣으면 어떤 ID값에 해당하는가를 알려주는 것을 Test라 한다.

> ### Classifier Detail
- Classifier는 머신러닝 알고리즘의 핵심으로, Neural Network, Random Forest, AdaBoost 등 수많은 알고리즘이 있다.
- 일반적으로 Classifier를 트레이닝 하는데 많은 양의 Data가 필요한데 이러한 수많은 Data를 수집하는 것은 한계가 있다.
- 따라서, 한정된 정보의 Database로 Training 시키기 위해서 Cross Validation 기법이나 Bootstrap 기법이 주로 사용된다.
- Cross Validation은 Database 중 일부를 이용하여 Training Set으로 정하되, 그 일부를 바꿔가며 Training and Test 하는 기법이다.
- Bootstrap은 Training Set을 선택하는데 있어, 정해진 값을 선택하는 것이 아니라, 해당 값의 확률 분포를 Modeling하고 그 모델에서 원하는 - 만큼의 데이터를 뽑아서 사용하는 것이다.

---

## 모델

### Neural Network

![image](https://github.com/lasiyan/lasiyan.github.io/assets/135001826/72fbb465-d56d-4421-afff-6e5a27bc56e6)

#### structure
- 뉴럴 네트워크는 X1 ~ Xn까지의 적절한 개수의 입력과 Bias(여기서 +1)가 input으로 들어온다.
- 이러한 입력값들은 더해진 후 Activation Function을 통해 output으로 출력된다.
- 위 그림과 같이 input layer와 output layer로 이루어진 뉴럴 네트워크 구조를 perceptron이라 한다.

#### linear Classifier: OR Classification
input이 (0,0) 이면 output은 -1, input이 (0,1), (1,0), (1,1) 이면 output은 1

![image](https://github.com/lasiyan/lasiyan.github.io/assets/135001826/df86674e-9f93-4329-8858-c74367f255b3)

- (x1, x2)에 대한 (0,0), (0,1), (1,0), (1,1)의 경우를 생각해보면
- (0,0) 일 경우: 1 * (-0.5) + 0 * 1 + 0 * 1 = -0.5 -> 0 보다 작다 =  -1
- (0,1) 일 경우: 1 * (-0.5) + 0 * 1 + 1 * 1 = 0.5 -> 0 보다 크다 =  1
- (1,0) 일 경우: 1 * (-0.5) + 1 * 1 + 0 * 1 = 0.5 -> 0 보다 크다 =  1
- (1,1) 일 경우: 1 * (-0.5) + 1 * 1 + 1 * 1 = 1.5 -> 0 보다 크다 =  1

![image](https://github.com/lasiyan/lasiyan.github.io/assets/135001826/ed31b9d4-1e78-46a6-965a-7742ae9d1b35)

#### Limitation: XOR Classification
마찬가지로 XOR에 대한 결과를 Graph로 나타내면 다음과 같다.

![image](https://github.com/lasiyan/lasiyan.github.io/assets/135001826/9993657e-ec94-4df7-b031-fb3799d11d79)

- a문제는 기존의 perceptron은 하나의 선형 그래프로 표현되는 판별식을 갖는데 반해, XOR의 경우 두 개의 선을 통해 나타내지고, 다시 말해 perceptron으로 표현이 불가능하다.
- 따라서 Multi-Layer Perceptron 이라는 새로운 Neural Network가 제안된다.

#### Multi-Layer Perceptron
- Input layer 와 Output layer만을 가지는 Perceptron으로는 XOR이라는 간단한 문제를 푸는 것에도 한계가 생김
- 따라서 Hidden Layer를 두어 보다 복잡한 문제를 풀 수 있도록 한 것이 Multi-Layer Perceptron
- Multi-Layer Perceptron의 문제는 어떻게 수많은 뉴럴 네트워크의 weight 값을 갱신할 수 있는가? 이고 이를 해결하는 대표적인 방법은 Error - Back-Propagation 알고리즘이 있다.
- 이는 전형적인 Gradient Descent 알고리즘 하나이다.

Neural Network는 output signal을 얻기 위해 step function(계단 함수)이라는 activation function(활성 함수)이 들어가는데, step function은 불연속 함수이므로 미분이 되지 않는다.

따라서 Gradient Descent 알고리즘이 적용하는데 문제가 발생하고, 그러므로 Error Back-Propagation 알고리즘에서는 step function 대신 sigmoid function을 사용한다.


### Support Vector Machine
- Machine Learning 알고리즘은 training set 데이터를 어떻게 나눌 것인지 그 rule을 결정
- 그리고 rule을 일반화하여 다른 test set 데이터들에도 적용
- 다른 test set에도 그 rule이 잘 적용되면 일반화가 잘 되어있다 말하는데
- 일반화의 관점에서 Neural Network는 최적의 솔루션을 제공하지 못함

![image](https://github.com/lasiyan/lasiyan.github.io/assets/135001826/09637533-73f6-4f65-a731-23e4478180a8)

위 그림에서 선 A와 B중 더 좋은 판별식은 직선 B일 것이다. A와 B 모두 training set은 완벽하게 분리할 수 있지만, 추후 test data를 판별하고자 할 때 A는 B에 비해 많은 오류를 야기할 가능성이 높기 때문이다.

그리고 이 최적의 선 B를 기준으로 가장 가까운 두 Data에 해당하는 두 점 Xsv.1, Xsv. 2을 support vector라 한다.


### Kernel SVM

Perceptron의 XOR 문제와 같이 SVM에서도 리니어한 하나의 평면으로 데이터셋을 구분하기 때문에 해당 문제를 풀 수 없다.

따라서 뉴럴 네트워크에서는 히든레이어를 추가하여 해당 문제를 해결한 것과 유사하게 SVM에서는 차원을 늘리는 방법을 사용한다.

하지만 이렇게 공간상의 매핑을 하지 않고도 비슷한 효과를 낼 수 있는 방법이 있는데 그건 Kernel trick을 적용하는 것이다. 기존의 SVM은 support vector와의 거리를 최대화하는 hyperplane을 찾는 방법인데 반해 Kernel SVM은 단순히 거리를 사용하는 것이 아닌 거리에 Kernel 함수를 통과시켜 사용한다.

> 출처: [http://www.whydsp.org/237](http://www.whydsp.org/237){:target="_blank"}