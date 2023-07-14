---
title: YOLOv5 실습하기 - Jetson Xavier NX
date: 2022-05-16 14:23:00 +0900
categories: [Stack, A.I]
tag: [AI, YOLO, Xavier NX, Anaconda]
---

YOLOv5 테스트를 위하여 Xavier NX 보드에서 Anaconda와 PyTorch, YOLOv5를 설치하고 이를 실행하는 방법을 정리한 글입니다.

> 아나콘다는 일반 AMD64 계열의 경우 linux 버전도 지원하지만, 라즈베리와 같은 ARM 계열에 대한 지원을 찾을 수 없어 유사한 플랫폼을 사용했습니다.

### 개발 환경
- Hardware : Nvidia Jetson Xavier NX (aarch64)
- OS : Jetpack 4.6 rev. 3 (based on Ubuntu 18.04)


### 아나콘다 설치
Jetson Anaconda로 검색해보면 Anaconda를 aarch64 버전으로 빌드한 것들이 많았는데, Mini-forge가 최근 업데이트가 활발한 것으로 보였고, 이를 선택했습니다.

[https://github.com/conda-forge/miniforge/releases](https://github.com/conda-forge/miniforge/releases){:target="_blank"}

> 위 글에 사용된 버전은 `Miniforge3-4.12.0-0` 입니다.

```bash
# 다운로드
wget https://github.com/conda-forge/miniforge/releases/download/4.12.0-0/Mambaforge-4.12.0-0-Linux-aarch64.sh

# 권한 변경
sudo chmod 777 Mambaforge-4.12.0-0-Linux-aarch64.sh

# 설치
sudo ./Mambaforge-4.12.0-0-Linux-aarch64.sh

# 환경 변수 등록
vi ~/.bashrc

# ... 파일 맨 마지막에 추가 후 :wq(저장) 입력
export PATH=콘다_설치_경로/bin:$PATH

# 저장 후 재실행
source ~/.bashrc

# 동작 확인
conda --version # 버전이 정상적으로 출력되는지 확인
```


### 개발 환경 구축
YOLO 실습을 `darknet`이라는 이름의 가상 환경을 생성하고 이를 활성화 하는 방법을 다룹니다. 가상 환경은 동작의 안정성과 타 프로젝트 영향력을 최소화하기 위하여 사용합니다.

> YOLOv5인데 Darknet인 이유는 이전 테스트했던 것이 YOLOv4이기 떄문입니다..

```bash
# darknet 이라는 환경을 생성한다. 이것은 파이썬 3.8 버전으로 동작된다.
conda create -n darknet python=3.8

# 가상 환경을 활성화한다.
conda activate darkent
```

생성에 성공하면 기본 파이썬 패키지들이 설치되었다고 출력됩니다. 그리고 아나콘다를 설치한 시점부터 터미널 맨 앞 접두사로 `(base)`가 보입니다. 이것은 아나콘다를 설치 시 출력되는데, 현재 활성화된 환경이 base(초기 상태)라는 것을 의미합니다.

그리고 우리가 가상 환경을 활성화(activate)하면 base가 `darknet`으로 변경됩니다.

우리가 향후 설치하는 모든 파이썬 패키지는 현재 활성화되어 있는 환경에 설치됩니다.

가상 환경에 대한 자세한 설명은 아래 경로에 나와 있습니다.

> 참고: [https://arcdocs.leeds.ac.uk/software/compilers/anaconda.html](https://arcdocs.leeds.ac.uk/software/compilers/anaconda.html)


예를들어 동일한 파이썬이라도 아래와 같이 활성화된 환경이 다르면, 각기 다른 버전이 출력됩니다.

![image](https://github.com/lasiyan/lasiyan.github.io/assets/135001826/f228571a-fc7b-4177-9893-79c7b85579f2)


### PyTorch 설치
일반적인 환경이라면 아나콘다를 통해 간단하게 PyTorch 및 연관 패키지 설치가 가능합니다.

그러나 제가 테스트하는 환경은 ARM64, 다시말해 aarch64 환경입니다. 그러나 이 환경에서 해당 패키지를 설치하기 위한 로컬 저장소가 없어 pip 패키지를 통해 설치하였습니다.

```bash
conda activate darknet
pip install torch
```

### YOLOv5 설치
```bash
git clone https://github.com/ultralytics/yolov5 # download YOLOv5

cd yolov5 # move to project folder

pip install -r requirements.txt # install package
```


### 동작 확인
패키지가 제대로 설치되었는지 확인하는 간단한 방법입니다.

```
# YOLOv5 샘플 실행 (yolov5 폴더 내)
python detect.py
```

처음 실행할 경우, weights 파일이 없기 때문에 yolov5s.pt 파일을 다운로드 진행할 것이고, 기타 모든 옵션은 기본값을 토대로 진행됩니다. 

따라서 위 커맨드는 `yolov5/data/images` 폴더 안에 있는 2장의 샘플 이미지를 토대로 객체 검출을 진행하며, 실행 결과는 yolov5/runs/detect/exp 폴더 안에 생성됩니다.

![image](https://github.com/lasiyan/lasiyan.github.io/assets/135001826/6c96b1c8-6e39-47e4-8048-5b467938b15d)
