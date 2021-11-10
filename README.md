# DACON-Traffic-Hand-Signals
## [데이콘] 2021 교통 수(手)신호 동작 인식 AI 경진대회 7등
<br>

## 📌 대회 설명 
1. 개요 : 교통 수(手) 신호 패턴 영상에서 추출한 이미지 학습 데이터를 활용한 인공지능 모델 기반의 교통 수신호 동작 인식 모델 개발
2. 평가 기준 : Log Loss 
<br>

## 📌 문제 접근

**✔ keypoints 분석** <br>
1. 각 이미지별 메타 데이터(json 파일)에 제공되는 keypoints 정보를 활용하기 위하여 각 keypoints가 어떤 부위에 해당하는지 구분하기 위한 분석 진행
2. 각 이미지에 keypoints 좌표를 표시하고 bounding box로 crop하여 시각화
3. 임의의 이미지들을 시각화해본 결과, 제공된 keypoints가 각 신체 부위에 정확히 위치해있기 때문에 이 정보를 충분히 활용할 수 있다고 판단
4. 제공된 keypoints는 각 이미지당 총 24개가 존재하였으며 각각 다음의 신체 부위에 대응됨. 
   > 0 : 'left_ankle'/ 1 : 'left_foot' / 2: 'left_elbow' / 3: 'left_hip' / 4: 'left_knee' / 5: 'left_shoulder' / 6: 'left_wrist' / 7: 'sacrum' / 8: 'right_ankle' / 9: 'right_foot' / 10: 'right_elbow' / 11: 'right_hip' / 12: 'right_knee' / 13: 'right_shoulder' / 14: 'right_wrist' / 15: 'waist' / 16: 'back' / 17: 'neck' / 18: 'forehead' / 19: 'left_ear' / 20: 'left_eye' / 21: 'nose' / 22: 'right_ear' / 23: 'right_eye' 
<br>

**✔ keypoints features 전처리**<br>
1. keypoints 좌표값을 바탕으로 특정 관절 부위간의 거리 계산
2. 교통 수신호 동작에서 '팔과 손 부위의 위치가 어떻게 변하느냐'가 해당 동작을 구분하는 데에 중요한 요소라고 판단하여 keypoints 중 elbow와 wrist 부위에 중점을 두었음. 따라서 각 시간축에 따라 다른 주요 신체 부위(shoulder, hip, ear)와 elblow, wrist간의 거리를 계산하여 동작을 수행하면서 팔과 손 부위가 다른 신체 부위와 얼마나 가까워지고 멀어지는지에 대한 상대적인 변화 정도를 고려함.
<br>


**✔ 데이터 reshape**<br>
1. 각 file은 하나의 수신호 동작을 보여주는 동영상으로 간주할 수 있으며, 각 file에 들어간 이미지들이 동영상을 구성하는 프레임이라고 볼 수 있음. 이 때, LSTM 모델에서 각 프레임의 정보를 time step으로 하여 학습할 수 있도록 모든 file별로 이미지(프레임) 개수가 같게끔 reshape함.
2. 예를 들어 file별 이미지 개수가 가장 적은 경우가 141개였고 가장 많은 경우가 194개였는데 가장 적은 경우인 141개 프레임에 맞춰서 그 개수를 초과하는 프레임을 가진 경우에는 뒷부분을 절삭함. 절삭 시 뒷부분을 선택한 이유는, 각 file별로 완성된 동작이 2회 이상씩 반복되고 있었기 때문에 뒷부분 프레임을 조금 자르더라도 앞에서 완성된 동작을 학습할 수 있을 것이라는 판단 하에 진행
<br>

**✔ 모델링**<br>
1. Bidirectional LSTM 모델 사용 : 시간축의 양방향으로 학습하여 이전 동작의 정보와 이후 동작의 정보를 모두 고려할 수 있게끔 하기 위함
2. StratifiedKFold를 사용하여 4개의 validation fold sets으로 나누었고 for loop를 통해 각 fold set에 대하여 모델링을 적용하여, 각각 다른 데이터 구성으로 총 4번의 학습과 추론 결과를 블렌딩 앙상블해주는 효과
3. n_splits=4, epochs = 200, patience = 20으로 설정했을 때 모델 학습 및 inference까지 약 2분 소요(Tesla P100 기준) 및 validation loss 0.47대
4. n_splits=8, epochs = 5000, patience = 300으로 설정했을 때 모델 학습 및 inference까지 약 22분 소요(Tesla P100 기준) 및 validation loss 0.11대
