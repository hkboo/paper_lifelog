# 이질적 보조분류기를 활용한 라이프로그 데이터 기반 모델링 방법론

## 실험 환경
- Colab
  - `Python` : `3.9.16`
  - `keras` : `2.12.0`
  - `tensorflow` : `2.12.0`
  - `GPU` : `NVIDIA A100`
    - **하드웨어 가속기에서 `GPU` 선택 필수**

- Local
  - 아나콘다 환경 (파이썬 버전 무관)
  - 1차 가공에 이용됨


## 데이터세트
- ETRI 라이프로그 데이터세트
  - https://nanum.etri.re.kr/share/schung1/ETRILifelogDataset2020?lang=ko_KR
    - user01-06 data
    - user07-10 data
    - user11-12 data
    - user21-25 data
    - user26-30 data
  - 해당 데이터를 활용해 가공하여 모델 학습 및 평가에 사용함

## 데이터 전처리
- 1차 가공 : 위 사용자별 라이프로그 데이터세트를 합친 1개의 결과를 도출하는 과정
  - (Local) data_handing/get_user_data.ipynb 
  - 적용 로직
    - 사용자별 label.csv이 없는 경우 제외
    - 행 중복 제거
    - ts 컬럼으로부터 파생 변수 생성
      - 년, 월, 일, 오전/오후 여부, 주말여부 등
  - 전체 사용자를 합친 1차 가공 결과는 data_handing/outputs/all_users_data.csv 로 저장됨

- 2차 가공 : 1차 가공된 데이터를 기반으로 실제 모델에 이용될 입/출력 데이터세트를 도출하는 과정
  - (Colab) 첨부 코드 1. 데이터 전처리
  - 1차 가공된 데이터는 깃에 업로드되어 있으므로 업로드된 데이터를 이용해 Colab에서 2차 가공 수행함
  - 적용 로직
    - 결측행 제거
      - `place` 컬럼내 null 값 1건 제거
      - 전체 402,877건 -> 402,876건으로 축소
    - 언더 샘플링(Under Sampling)
      - `place` 컬럼에서 `restaurant` 값이 가장 적음(18,735건)
      - 전체 402,876건 -> 93,675건으로 축소
      - `Seed` : `1004`
    - 감정, 흥미도 컬럼 생성
      - `emotionPositive` -> `emotionPositive_class`
        - 1, 2 -> `부정`, 3, 4, 5 -> `중립`, 6, 7 -> `긍정`
      - `emotionTension` -> `emotionTension_class`
        - 1, 2 -> `차분`, 3, 4, 5 -> `흥미`, 6, 7 -> `흥분` 
    - 원-핫 인코딩(One-hot Encoding) 수행
    - 데이터 셋 분할
      - `T/V/T` : `6:2:2`
      - `Seed` : `1004`



## 모델 학습 및 평가
- 비교, 제안 모델 학습 및 평가

  - 첨부 코드 2. 모델 학습 및 평가 (Colab)
  - 데이터 전처리 과정에서 `2차 가공된 데이터(data_undersampled.csv)`를 이용하여 모델 학습 및 평가를 수행함
  - 즉, 비교 모델과 제안 모델의 정확도 비교하고자 함
  - `Epochs`와 `Loss Weight`를 제외한 나머지 파라매터는 모두 고정하였으며 적용 파라매터는 다음과 같음
    - 적용 파라매터
      - `Epochs` : `10`, `20`, `100`
      - `Loss Weight`: `0`, `0.1`, `0.2`, `0.3`, `0.4`, `0.5`, `0.6`, `0.7`, `0.8`, `0.9`, `0.1` (보조 분류기 적용 모델에 이용됨)
      - `Early Stopping` : `monitor='val_loss', mode='min', patience=2` (`Epochs`가 `100`일 때 적용됨)
      - `Batch Size` : `128`
      - `Activation` : `relu`
      - `Optimizer` : `adamax`
      - `Seed` : `1004`


## 성능 평가 결과
**이질적 보조분류기를 적용한 모델이 적용하지 않은 모델보다 정확도가 향상됨을 확인**

#### (1) Epochs = 10
|        **Model type**       	| **Epochs** 	| **Loss Weight** 	| **Accuracy** 	| **Macro F1** 	|
|:---------------------------:	|:----------:	|:---------------:	|:-------:	|:------------:	|
| 비교모델1_보조분류없음      	|     10     	|        0        	|  70.79% 	|    70.36%    	|
| 비교모델2_보조분류_장소     	|     10     	|        1        	|  71.64% 	|    71.43%    	|
| 제안모델1_보조분류_감정     	|     10     	|       0.5       	|  71.69% 	|    71.45%    	|
| 제안모델2_보조분류_흥미도   	|     10     	|       0.7       	|  71.64% 	|    71.54%    	|
| 제안모델3_보조분류_활동상세 	|     10     	|       0.7       	|  **72.01%** 	|    **71.66%**    	|

#### (2) Epochs = 20
|        **Model type**       	| **Epochs** 	| **Loss Weight** 	| **Accuracy** 	| **Macro F1** 	|
|:---------------------------:	|:----------:	|:---------------:	|:-------:	|:------------:	|
| 비교모델1_보조분류없음      	|     20     	|        0        	|  71.13% 	|    70.24%    	|
| 비교모델2_보조분류_장소     	|     20     	|       0.5       	|  **72.58%** 	|    71.92%    	|
| 제안모델1_보조분류_감정     	|     20     	|       0.5       	|  72.31% 	|    **72.09%**    	|
| 제안모델2_보조분류_흥미도   	|     20     	|       0.4       	|  72.15% 	|    71.90%    	|
| 제안모델3_보조분류_활동상세 	|     20     	|       0.6       	|  72.32% 	|    71.84%    	|

#### (3) Epochs = 100, Early stopping 적용(실험 파라매터 참조할 것)
|        **Model type**       	| **Epochs** 	| **Loss Weight** 	| **Accuracy** 	| **Macro F1** 	|
|:---------------------------:	|:----------:	|:---------------:	|:-------:	|:------------:	|
| 비교모델1_보조분류없음      	|   100-ES   	|        0        	|  72.63% 	|    72.33%    	|
| 비교모델2_보조분류_장소     	|   100-ES   	|       0.2       	|  72.71% 	|    72.21%    	|
| 제안모델1_보조분류_감정     	|   100-ES   	|       0.1       	|  72.71% 	|    **72.48%**    	|
| 제안모델2_보조분류_흥미도   	|   100-ES   	|       0.7       	|  **72.78%** 	|    72.23%    	|
| 제안모델3_보조분류_활동상세 	|   100-ES   	|       0.8       	|  72.76% 	|    72.27%    	|

## 코드 이용 방법
**1. 원천 데이터 처리**
  - data_handing/get_user_data.ipynb
  - 데이터의 용량이 크므로 해당 과정은 Colab을 이용하지 않고 로컬에서 1차 가공 과정을 수행함
  - 일부 패키지 설치 필요 (이 때, 파이썬 버전 무관함)
    - `conda install -c conda-forge pandas`
  - 이 때, 1차 가공을 위한 라이프로그 데이터셋의 자세한 경로는 아래 캡처를 참조할 것
    ![경로확인](https://user-images.githubusercontent.com/66122975/231943075-e329cfc5-dc04-4958-be45-0cba18586ee0.png)

**2. 데이터 전처리**  [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1et6TvdwUNq8Q8PNjQnMJk7cZLi_Pcwbh?usp=sharing)
  - Colab을 이용하므로 패키지 설치 불필요함
  - 본 실험에서는 모델 과적합을 피하기 위해 언더 샘플링을 수행함
    - 위 첨부 코드에서 `do_undersampling` = `True`로 설정되어 있음
    - `do_undersampling` = `False`로 설정한다면 언더 샘플링을 수행하지 않음
  - 2차 가공 결과는 코랩 환경에서의 디스크에 저장되며 해당 데이터 파일을 따로 저장하는 것이 필요함
    - **본 실험을 위해 결과를 paper_lifelog/main/outputs에 데이터를 업로드하였으며 해당 경로에 있는 데이터를 모델 학습 및 평가에 이용함**
      - https://raw.githubusercontent.com/hkboo/paper_lifelog/main/outputs/data_undersampled.csv

**3. 모델 학습 및 평가**  [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1IH27LkiT3BtSZRFiSc6JimiSGiSALQXh?usp=sharing)
  - Colab을 이용하므로 패키지 설치 불필요함
  - 자동으로 산출물 저장가능케 설정함
    - 본인의 산출물 경로가 있는 경우 `MODEL_OUTPUT_PATH` 변수에 경로 지정할 것
  - 위 첨부 코드에서 `EPOCH_N` 변수에 원하는 Epochs을 지정할 것
    - 본 실험에서는 `10`, `20`, `100`을 지정해 실험 수행하였음
  - 최종 산출물은 지정된 `MODEL_OUTPUT_PATH` 경로 내 파일명 `전체정확도.csv`로 저장됨
    - 중간 각 모델별로 loop별 결과를 누적해 저장될 수 있도록 함 (함수 `save_result(control_type, result_dataframe)`)
  - 해당 코드는 `Epochs = 2`일 때의 결과를 포함하는 예시임
