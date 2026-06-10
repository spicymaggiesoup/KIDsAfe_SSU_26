# KIDsAfe — 어린이보호구역 위험도 예측 AI

<img width="300" alt="kidsafe_logo" src="https://github.com/user-attachments/assets/a62db558-d60a-45b1-81dc-f63d89311db3"/>

> **K**ids **I**ntelligent **D**anger **S**afety **A**I for **F**amily **E**nvironment  
> 전국 어린이보호구역 주변 7개 요인과 교통량을 분석해 교통사고 위험도를 예측하는 AI 서비스

🔗 **서비스 바로가기**: https://kidsafe-ssu-26.onrender.com/app

<img width="250" height="250" alt="qrcode_360077965_e01eb64a0dcf9435260bd8e172168213" src="https://github.com/user-attachments/assets/e7dd4a97-daa1-4aee-910b-ab4ce5c73aa6" />
<img width="550" height="250" alt="image" src="https://github.com/user-attachments/assets/c7693355-eac7-49cc-adde-306cfa5bee31" />

---

## 📌 프로젝트 개요

전국 어린이보호구역 **14,606개**를 대상으로, 반경 500m 내 **7개 시설물 요인**과 **교통량 지표**를 분석하여 해당 구역의 위험도를 **안전 / 주의 / 위험** 3단계로 예측합니다.


| 구분 | 요인 |
|---|---|
| 🚸 분석 요인 (7개) | 과속방지턱, 도로안내표지, 무인단속카메라, 보행자우선도로, 일방통행도로, 지역특화거리, 옐로카펫 |
| 🚦 교통량 지표 | 주차장 + 버스정류장 (StandardScaler 합산) |

---

## 🗂 파일 구조

```
KIDsAfe_SSU_26/
├── main.py              ← FastAPI 서버 (API 전체 로직)
├── index.html           ← 프론트엔드 (HTML + Vanilla JS)
├── requirements.txt     ← Python 패키지 목록
├── render.yaml          ← Render.com 배포 설정
├── .gitignore
└── models/
    ├── best_model.pkl              ← 학습된 XGBoost 모델 (SMOTE 적용)
    ├── scaler.pkl                  ← MinMaxScaler (8피처)
    ├── traffic_scaler.pkl          ← StandardScaler (주차장·버스정류장)
    ├── zones.pkl                   ← 보호구역 지도 표시 데이터
    ├── tree_과속방지턱수.pkl          ← BallTree (반경 카운팅용)
    ├── tree_도로안내표지수.pkl
    ├── tree_무인단속카메라수.pkl
    ├── tree_보행자우선도로수.pkl
    ├── tree_일방통행도로수.pkl
    ├── tree_지역특화거리수.pkl
    ├── tree_옐로카펫수.pkl
    ├── tree_주차장수.pkl
    └── tree_버스정류장수.pkl
```

---

## 🤖 모델 정보

| 항목 | 내용 |
|---|---|
| **모델** | XGBoost (다중분류, 3단계) |
| **분류** | 안전(0): 사고 0건 / 주의(1): 발생·중앙값 이하 / 위험(2): 발생·중앙값 초과 |
| **학습 데이터** | 전국 어린이보호구역 14,606개 (사고 0건 대조군 10,545개 포함) |
| **반경** | 500m |
| **독립변수** | 8개 (시설물 7 + 교통량 지표 1) |
| **불균형 보정** | SMOTE (Train만 적용) |
| **CV F1 (weighted)** | 0.698 |
| **F1 (macro)** | 0.536 (SMOTE 적용, 기본 0.464 대비 개선) |

### 분석 방법론

세 가지 분석이 동일한 결론으로 수렴합니다 (방법론적 삼각측정).

| 단계 | 방법 | 결과 |
|---|---|---|
| 1단계 | 상관분석 | 7개 요인 전부 양의 상관 (예상과 반대) |
| 2단계 | 부분상관 | 교통량 통제 시 상관 약 50% 감소, 그러나 여전히 양(+) |
| 3단계 | 머신러닝 | 교통량 지표 중요도 1위 (0.269), 교통량 추가 시 전 모델 성능 향상 |

### 요인 중요도 (XGBoost)

| 순위 | 요인 | 중요도 |
|---|---|---|
| 1 | 교통량 지표 | 0.269 |
| 2 | 무인단속카메라 | 0.150 |
| 3 | 도로안내표지 | 0.143 |
| 4 | 과속방지턱 | 0.103 |
| 5 | 보행자우선도로 | 0.094 |
| 6 | 지역특화거리 | 0.091 |
| 7 | 일방통행도로 | 0.080 |
| 8 | 옐로카펫 | 0.071 |

---

## 🚀 API 명세

### `POST /predict`

위도/경도를 입력하면 위험도를 3단계로 예측합니다.

**요청**
```json
{
  "lat": 37.5665,
  "lng": 126.9780
}
```

**응답**
```json
{
  "lat": 37.5665,
  "lng": 126.9780,
  "위험도등급": 2,
  "위험도명": "위험",
  "안전확률": 0.1523,
  "주의확률": 0.2104,
  "위험확률": 0.6373,
  "features": {
    "과속방지턱수": 24,
    "도로안내표지수": 3,
    "무인단속카메라수": 8,
    "보행자우선도로수": 1,
    "일방통행도로수": 2,
    "지역특화거리수": 0,
    "옐로카펫수": 1,
    "주차장수": 15,
    "버스정류장수": 42,
    "교통량지표": 1.873
  }
}
```

| 엔드포인트 | 설명 |
|---|---|
| `GET /` | 서버 상태 확인 |
| `GET /health` | 헬스체크 |
| `GET /app` | 프론트엔드 UI |
| `GET /zones` | 보호구역 좌표 목록 (지도 표시용) |
| `POST /predict` | 위험도 예측 |
| `GET /docs` | Swagger UI |

---

## 🛠 로컬 실행

```bash
# 패키지 설치
pip install -r requirements.txt

# 서버 실행
uvicorn main:app --reload

# 접속
http://localhost:8000/app     ← 프론트엔드
http://localhost:8000/docs    ← Swagger UI
```

---

## 📊 데이터 출처

| 데이터 | 출처 |
|---|---|
| 전국 어린이보호구역 표준데이터 | 공공데이터포털 |
| 전국 교통사고 다발지역 표준데이터 | 공공데이터포털 |
| 전국 과속방지턱·도로안내표지·무인단속카메라 등 표준데이터 | 공공데이터포털 |
| 전국 주차장·버스정류장 위치정보 | 공공데이터포털 |

---

## 👥 팀 정보

**숭실대학교 AI프로그래밍 6조**

---

## 📄 License

MIT License
