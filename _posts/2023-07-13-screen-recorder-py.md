---
title: 화면 녹화 프로그램 - Python, OpenCV
date: 2023-07-13 16:45:00 +0900
categories: [etc., Utility]
tag: [화면 녹화, 파이썬, Python, OpenCV]
---

## 서론

요즘 TensorFlow 공부겸 파이썬을 공부 중입니다..

문득 과거 학부생 때 사용했던 OpenCV를 연동하여 이미지/영상 처리를 하는 예제들이 있길래, 함께 보면서 프로토타입으로 구현해본 화면 녹화 프로그램 샘플 코드를 만들어 보았습니다.

본문은 해당 코드에 대한 간략한 리뷰입니다. 

---

코드는 크게 3파트로 구성되어 있습니다.

## define.py

말 그대로 공통으로 사용되는 변수에 대한 선언을 모아 놓은 문서입니다.

```python
DEF_TITLE = "Recoder"

DEF_KEY_REC_START_UI = 'Alt+0'
DEF_KEY_REC_START = 'Alt+1'
DEF_KEY_REC_STOP = 'Alt+2'
DEF_KEY_SCREN_CAP = 'Alt+3'
```

해당 프로그램은 버튼이 아닌 Keyboard Event를 활용하여 기능이 실행되도록 구현하였습니다.


## func.py

레코드 및 캡처에서 사용되는 함수들을 포함한 모듈입니다.

핵심이 되는 Recording 함수는 다음과 같습니다.

```python
def StartRecord(root, roi, debug):

    global isRecord, output

    root.attributes('-alpha', 0.0)

    width = roi[2] - roi[0]
    height = roi[3] - roi[1]
    codec = cv2.VideoWriter_fourcc('m', 'p', '4', 'v')
    frame_rate = GetFramerate(roi)

    output = cv2.VideoWriter('outputput_%s.mp4' % GetFileName(), codec, frame_rate, (width, height))

    cnt = 1
    while output.isOpened():
        img = ImageGrab.grab(roi)
        frame = np.array(img)
        frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)

        if debug :
            cv2.imshow('Preview', frame)  # Display the picture

        output.write(frame)

        key = cv2.waitKey(1)

        bRecord = True

        print("recording%s" % ("." * (cnt % 10)))
        cnt+=1

        if keyboard.is_pressed(define.DEF_KEY_REC_STOP):
            StopRecord(root)
            break

    return

def StopRecord(root):
    global output

    cv2.destroyAllWindows()
    output.release()
    root.attributes('-alpha', 0.8)
    bRecord = False

    return
```

먼저 인자로 들어오는 `root`는 `Tkinter`를 활용하여 생성된 Dialog입니다.

이것을 함수로 전달하여 녹화 를 시작하면 화면에서 Dialog가 사라지도록 하였습니다. 기존에는 트레이 최소화를 생각하였으나,

MFC와 다르게 Python에서 Windows API 접근에 어려움이 있어 투명으로 변경하였습니다.

`roi`는 Dialog의 좌표입니다. L-T-R-B 순서로 roi 배열에 저장됩니다.

마지막으로 중간에 GetFramerate 함수가 있는데,

일단 해당 프로그램에선 동적으로 녹화 영역을 설정할 수 있도록 하였더니 FPS가 영역 크기에 따라 편차가 크게 발생하였습니다.

> 이는 녹화 영역에 따라 해상도 차이가 발생하여 I/O 시간 편차 때문에 발생하는 것으로 보입니다.

2560x1080 해상도 기준 13~16 fps로 계산되고, 사이즈가 작으면 30 fps까지도 안정적으로 출력되었습니다.

따라서 아래와 같이 대략적인 FrameRate를 구하도록 하였습니다.

```python
# Calc frame-rate from 10 frame's avg time (first frame was dropped)
def GetFramerate(roi=[]):
    sum = 0.0
    for i in range(11):
        last_time = time.time()
        frame = cv2.cvtColor(np.array(ImageGrab.grab(roi)), cv2.COLOR_BGR2RGB)
        cv2.waitKey(1)
        if not i == 0:  # drop first frame 
            sum += (1 / (time.time()-last_time))
        print('cal fps: {0}'.format(1 / (time.time()-last_time)))

    frame_rate = sum / 10
    print('Set Frame rate: %f' % frame_rate)

    return frame_rate
```

첫 프레임을 제외한, 총 10장의 프레임에서 평균 FPS를 산출합니다.

> 테스트한 환경에서 첫 프레임 값이 매우 불안정하였습니다.


## main.py

마지막으로 Dialog를 구성하는 UI 코드입니다.

```python
# Keyboard event
def SetFunc(event):

    global coordinate
    
    if keyboard.is_pressed(define.DEF_KEY_REC_START):
        threading.Thread(target=Func.StartRecord(root, coordinate, False), daemon=True).start()

    if keyboard.is_pressed(define.DEF_KEY_REC_START_UI):    #debugging
        threading.Thread(target=Func.StartRecord(root, coordinate, True), daemon=True).start()

    if keyboard.is_pressed(define.DEF_KEY_SCREN_CAP):
        threading.Thread(target=Func.Capture(root, coordinate), daemon=True).start()

    return

# Dialog re-size event
def SetSize(event):

    time.sleep(0.001)

    global coordinate
    coordinate = list()
    
    width = root.winfo_width()
    height = root.winfo_height()
    
    coordinate.append(root.winfo_rootx())   # x cord.
    coordinate.append(root.winfo_rooty())   # y cord.
    coordinate.append(coordinate[0] + width)
    coordinate.append(coordinate[1] + height)

    Title = define.DEF_TITLE + ' ' + str(width) + ' x ' \
         + str(height) + ' ' + str(coordinate[0]) + ' x ' + str(coordinate[1])

    root.title(Title)
    
    time.sleep(0.01)

root.bind("<Key>", SetFunc)
root.bind("<Configure>", SetSize)
```

bind 함수를 통해 이벤트를 설정하였습니다.

SetSize는 녹화 전 녹화할 영역을 선택하고 설정된 좌표(roi)를 func 모듈에 전달합니다.

일단 키가 입력되면 SetFunc를 호출하도록 하였는데 `<Key>` 모듈 사용 시 Shift+방향키 등은 동작하지만 기타 조합키가 정상적으로 동작되지 않아 `<keyboard>` 모듈을 사용하였습니다.

아래는 실행 영상이 포함된 유튜브 및 GitHub 경로입니다.

YouTube: [https://youtu.be/R-E4UvOoK0E](https://youtu.be/R-E4UvOoK0E)

GitHub: [https://github.com/lasiyan/Screen-Recorder](https://github.com/lasiyan/Screen-Recorder)