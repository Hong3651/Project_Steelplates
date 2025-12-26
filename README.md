# Project_Steelplates

후판(Ship Plate) 공정 데이터를 기반으로 **Scale(스케일) 불량을 사전 예측**하고,  
불량에 영향을 주는 핵심 공정 조건을 도출하여 **품질 안정화 + 공정 운영 의사결정**에 활용하는 머신러닝 프로젝트입니다.

---

## 1. 프로젝트 개요

### 목표
- 후판 공정 데이터로 **Scale 불량(이진 분류: 양품/불량)**을 예측
- 단순 예측 정확도보다, 현업 적용 관점에서 중요한:
  - **불량(Scale) Recall / F1-score** 중심으로 성능 최적화
- 예측 모델 결과를 해석해:
  - **불량 위험을 높이는 조건(리스크 구간)**
  - **관리 우선순위가 높은 변수(중요 변수)**
  를 도출

### 기대 효과(업무 관점)
- Scale 불량 발생 전 **사전 경보 및 리스크 플래그** 제공
- 공정 조건(온도/시간/치수/설비/디스케일링 등) 관리 포인트를 정량화하여
  - 품질 불량 비용 및 재작업 리스크 감소
  - 공정 조건 설정의 근거 강화

---

## 2. 파생변수(Feature Engineering)

공정 메커니즘을 반영하기 위해 다음 파생변수를 설계/추가했습니다.

- `shift_worker` : 작업조/근무조 추정(날짜/시간 기반)
- `fur_pre_time` : 가열로 총 시간에서 가열/균열 시간을 제외한 전단 시간
- `heat_soak_diff` : 가열온도 - 균열온도 차이(열 프로파일 특성)
- `pt_area` : 판 면적(`pt_width * pt_length`)
- `pt_vol` : 판 체적(`pt_area * pt_thick`)
- `steel_usage` : 규격/국가/스펙 기반 사용 분류(도메인 분류)

---

## 3. EDA 핵심 인사이트

- `hsb`:
  - **HSB 미적용 시 불량 비율이 매우 높게 관측(리스크 플래그 후보)**
- `rolling_temp`:
  - 특정 고온 구간(예: **950 이상**)에서 불량 비율 증가 경향 확인
- `fur_input_row`:
  - 단독 변수로는 불량/양품 차이가 크지 않아, 다른 변수와의 결합 영향 가능성 확인
- `heat_soak_diff`, `fur_total_time`:
  - 두 변수만으로는 경계가 명확하지 않아, **다변량 모델링 필요** 판단

---

## 4. 모델링 및 평가

### 평가 전략
- 목표가 “불량을 놓치지 않는 것”에 가까워:
  - Accuracy 단독보다 **불량(Scale) Precision / Recall / F1**을 핵심 지표로 사용
- Train/Test split 기반 평가 + Confusion Matrix로 오분류 성격 확인

### 실험 모델
- Decision Tree
- RandomForest
- GradientBoosting
- XGBoost (최종 채택)

---

## 5. 모델 실험 결과 요약

### (1) 피처 축소(연속형 위주) 시도
- 불량(Scale) F1-score가 **0.63 수준으로 낮게 관측**
- 결론: 범주형/공정 플래그성 변수 포함 및 파생변수 반영이 필요

### (2) 최종 비교(대표 결과)
- Decision Tree
  - Confusion Matrix: `[[206, 0], [2, 92]]` (오분류 2건 수준)
  - 불량 F1이 매우 높게 관측되나, 단일 트리 특성상 과적합 리스크 존재

- RandomForest
  - Confusion Matrix: `[[206, 0], [26, 68]]`
  - 불량 F1: **0.840**, Accuracy: **0.913**
  - 불량을 일부 놓치는 케이스(FN) 존재

- GradientBoosting
  - Confusion Matrix 기준 오분류 소수(약 3건 수준)
  - Accuracy 약 **0.99**, 불량 F1 약 **0.984** 수준

- ✅ XGBoost (Final)
  - Confusion Matrix: `[[206, 0], [1, 93]]`
  - **Accuracy: 0.997**
  - 불량(Scale) Precision: **1.000**
  - 불량(Scale) Recall: **0.989**
  - 불량(Scale) F1-score: **0.995**

---

## 6. 중요 변수 및 운영 포인트(해석)

XGBoost 중요도 기준 상위 변수:
- `rolling_temp`
- `hsb` (HSB 적용 여부)
- `fur_soak_temp`
- `pt_thick`, `pt_length` (치수/형상)
- `descaling_count`
- (그 외 가열로 시간/온도 관련 파생변수)

운영 관점 정리(예시):
- `rolling_temp` 고온 구간은 Scale 리스크 상승 가능 → 관리 기준선 설정 후보
- `hsb` 미적용 상태는 고위험 신호로 활용 가능 → 공정 조건/설비 상태 점검 트리거
- `fur_soak_temp` 및 체류시간 계열은 열이력의 안정성 지표로 관리 가능
- `descaling_count`는 품질과 비용(자원) 모두에 영향 → 예측 기반 최적화 후보

---



## 7. 기술 스택(Tech Stack)

- Language: Python
- Data: pandas, numpy
- Modeling: scikit-learn, XGBoost
- Visualization: matplotlib
- Collaboration: Git, PPT 기반 공유

---

## 8. 실행 방법(재현)

1) 환경 설치
- `required.txt` 기반 설치(required.txt)

2) 노트북 실행
- EDA → 전처리 → 파생변수 생성 → 모델 학습/평가 순서로 실행

---

## 11. 프로젝트 정보

- Program: POSCO AI Big Data Academy
- Topic: 후판(Ship Plate) 공정 Scale 불량 예측
- Output:
  - 모델 성능 비교 리포트
  - 최종 XGBoost 분류기 + 중요 변수 기반 운영 인사이트
