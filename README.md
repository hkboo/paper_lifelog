# paper_lifelog

## 데이터셋
ETRI 라이프로그 데이터셋
  - https://nanum.etri.re.kr/share/schung1/ETRILifelogDataset2020?lang=ko_KR
    - user01-06 data
    - user07-10 data
    - user11-12 data
    - user21-25 data
    - user26-30 data

## 데이터 전처리
1) 1차 가공
2) 2차 가공


## 모델 학습 및 평가
- 2차 가공된 데이터를 이용하여 모델 학습 및 평가 수행
- 즉, 비교 모델과 제안 모델의 정확도 비교하고자 함
- Epoch를 제외한 나머지 파라매터는 모두 동일하게 설정하였으며 사용 파라매터는 다음과 같음
  - 사용 파라매터
    - `Epoch` : `10`, `20`, `100`
    - `Early Stopping` : `monitor='val_loss', mode='min', patience=2`
    - `Batch Size` : `128`
    - `Activation` : `relu`
    - `Optimizer` : `adamax`

- 코드 사용 방법
  - 코랩을 이용하여 패키지 설치 불필요
  - 자동으로 산출물 저장가능케 설정 (본인의 산출물 경로가 있는 경우 경로 지정 가능)
    - `MODEL_OUTPUT_PATH` 변수에 경로 지정
  - 하단 첨부 코드에서 `EPOCH_N` 변수에 원하는 Epoch을 지정
    - 본 실험에서는 `10`, `20`, `100`을 지정해 실험 수행
  - 최종 산출물은 지정된 `MODEL_OUTPUT_PATH' 경로 내 파일명 `전체정확도.csv`로 저장됨
    - 중간 각 모델별로 loop별 개별 결과를 누적해 저장될 수 있도록 함 (함수 `save_result(control_type, result_dataframe)`)

## 성능 평가 결과
|        **Model_type**       	| **Epochs** 	| **Loss_weight** 	| **Acc** 	| **Macro F1** 	|
|:---------------------------:	|:----------:	|:---------------:	|:-------:	|:------------:	|
| 비교모델1_보조분류없음      	|     10     	|        0        	|  70.79% 	|    70.36%    	|
| 비교모델2_보조분류_장소     	|     10     	|        1        	|  71.64% 	|    71.43%    	|
| 제안모델1_보조분류_감정     	|     10     	|       0.5       	|  71.69% 	|    71.45%    	|
| 제안모델2_보조분류_흥미도   	|     10     	|       0.7       	|  71.64% 	|    71.54%    	|
| 제안모델3_보조분류_활동상세 	|     10     	|       0.7       	|  72.01% 	|    71.66%    	|
| 비교모델1_보조분류없음      	|     20     	|        0        	|  71.08% 	|    70.50%    	|
| 비교모델2_보조분류_장소     	|     20     	|       0.3       	|  72.63% 	|    72.05%    	|
| 제안모델1_보조분류_감정     	|     20     	|       0.9       	|  72.29% 	|    71.45%    	|
| 제안모델2_보조분류_흥미도   	|     20     	|       0.4       	|  72.43% 	|    71.91%    	|
| 제안모델3_보조분류_활동상세 	|     20     	|       0.6       	|  72.43% 	|    71.91%    	|
| 비교모델1_보조분류없음      	|   100-ES   	|        0        	|  72.63% 	|    72.33%    	|
| 비교모델2_보조분류_장소     	|   100-ES   	|       0.2       	|  72.71% 	|    72.21%    	|
| 제안모델1_보조분류_감정     	|   100-ES   	|       0.1       	|  72.71% 	|    72.48%    	|
| 제안모델2_보조분류_흥미도   	|   100-ES   	|       0.7       	|  72.78% 	|    72.23%    	|
| 제안모델3_보조분류_활동상세 	|   100-ES   	|       0.8       	|  72.76% 	|    72.27%    	|
- 보조분류기를 적용한 모델이 적용하지 않은 모델보다 정확도가 향상됨을 확인할 수 있음

## [첨부] 코드
1. 데이터 전처리<br>
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1et6TvdwUNq8Q8PNjQnMJk7cZLi_Pcwbh?usp=sharing)


2. 모델 학습 및 평가<br>
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1IH27LkiT3BtSZRFiSc6JimiSGiSALQXh?usp=sharing)
