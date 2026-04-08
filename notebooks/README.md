# Notebooks 디렉토리

> 이 디렉토리는 EDA·가설 검정·모델링 전 과정을 담은 노트북 5개와 전 과정을 통합한 최종본 1개로 구성된다.  
> **`05_integrated_analysis.ipynb`가 최종 통합본**이며, 개별 노트북은 역할별 단위 파일이다.

---

## 파일 구성 및 실행 안내

| 파일 | 역할 | 독립 실행 |
|---|---|---|
| `01_가설검증_0330.ipynb` | 글로벌 분쟁 데이터 병합 + Granger 인과 검정 | ✅ |
| `02_1_middle_east_eda_otto.ipynb` | ACLED 중동 데이터 EDA (CAT 분류 방식) | ✅ |
| `02_2_middle_east_eda_teammate.ipynb` | ACLED 중동 데이터 EDA (지역 그룹 분류 방식) | ✅ |
| `03_modeling_v4.ipynb` | 분위 회귀 & LightGBM Quantile (V1~V4) | ✅ |
| `04_modeling_v9.ipynb` | GARCH-X Monte Carlo 최종 모델 (V9) | ✅ |
| `05_integrated_analysis.ipynb` | **전 과정 통합본** — 위 5개 노트북의 핵심 로직을 순서대로 재현 | ✅ |

> `05_integrated_analysis.ipynb`만 실행해도 전체 분석 흐름을 재현할 수 있다.
> > `05_integrated_analysis.ipynb` 노트북은 `data/dataset_v28.csv`만 있으면 독립 실행 가능하다.  

---

## 노트북별 상세 설명

---

### `01_가설검증_0330.ipynb`
**글로벌 분쟁 데이터 통합 및 가설 3·4 검정**

EDA 단계에서 중동 데이터만 분석했을 때의 한계를 보완하기 위해, ACLED 글로벌 데이터(아프리카·아시아-태평양·유럽·미주 등 6개 권역)를 병합하고 중동 결과와 비교 검정한다.

**수행 내용**:
- 권역별 ACLED xlsx 파일 → csv 변환 후 통합 병합
- 글로벌 CAT별 GPR 상관관계 히트맵 (중동 결과와 비교)
  - CAT1 ↔ GPR_ACT: 중동 **+0.244** → 글로벌 **+0.038** (급락 확인)
  - 원인: 글로벌 분쟁 상당수가 산유 공급로와 무관한 지역에서 발생
- **Granger 인과 검정** (GPR_THREAT → Brent, Lag 1~30일)
  - 전 구간 p-value ≥ 0.05 → GPR_THREAT의 선행 인과 관계 미확인
  - GPR_THREAT는 유가 방향 예측자가 아닌 **동시 반응 지표**임을 통계적으로 확인

**핵심 결론**:

| 가설 | 결과 |
|---|---|
| 분쟁 빈도·사망자 수가 GPR에 영향을 줄 것이다 | 기각 |
| 분쟁 발생과 유가 변동 사이에 시차가 있을 것이다 | 기각 (전 구간 p≥0.05) |

---

### `02_1_middle_east_eda_otto.ipynb`
**ACLED 중동 분쟁 EDA — 이벤트 카테고리(CAT) 분류 방식**

ACLED 중동 집계 데이터를 **분쟁 성격** 기준 4개 카테고리로 분류하고, 카테고리별로 GPR 지수와의 상관 구조를 분석한다.

**분석 목차** (10개 섹션):
1. 라이브러리 & 데이터 로드
2. 기본 데이터 탐색 (컬럼·타입·기술통계)
3. 결측값 처리
4. 연도별 추이 분석
5. 국가별 분석
6. 사건 유형(EVENT_TYPE) 분석
7. **분쟁 카테고리 분류** ← 핵심
8. 최근 동향 (2024~2026)
9. 지리적 분포 시각화
10. 상관관계 분석

**CAT 분류 체계**:

| CAT | 정의 | 건수 |
|---|---|---|
| CAT1 | 고강도 군사·무기 공격 (Armed clash, Air/drone strike, Shelling, Suicide bomb 등) | 269,464건 |
| CAT2 | 비정규 폭력·인프라 파괴 (Attack, IED, Looting 등) | 48,383건 |
| CAT3 | 사회적 소요·민간 불안 (Protests, Mob violence 등) | 54,913건 |
| CAT4 | 정치·지정학적 전환 (Territory change, Agreement 등) | 8,772건 |

**핵심 발견**:
- CAT1(고강도 군사)만 GPR_ACT와 유의미한 양의 상관 (+0.244)
- CAT4(영토 변화)는 음의 상관 (-0.171) — 전쟁 종료·안정화 시점과 동시 발생하는 구조적 이유

---

### `02_2_middle_east_eda_teammate.ipynb`
**ACLED 중동 분쟁 EDA — 지역 그룹(GROUP) 분류 방식**

동일한 중동 ACLED 데이터를 **지리적·전략적 중요도** 기준으로 그룹화하여, 호르무즈 해협 인접성이 유가 영향에 실질적 차이를 만드는지 분석한다.

**그룹 분류 체계**:
- **GROUP_3**: Iran / OPEC / Other (3분류)
- **GROUP_4**: Iran / OPEC_Hormuz / Non_OPEC_Hormuz / Other (호르무즈 인접성 추가 반영)

**분석 과정**:
- 결측치 처리 (전체 행 NaN 제거 후 csv 저장)
- 데이터 탐색 (shape, dtypes, info)
- 결측치 파악 및 보완
- 그룹별 이벤트·사망자 분포 비교
- POPULATION_EXPOSURE 결측치 회귀 보완 (M1 raw OLS → M2 log-y OLS → 최종 채택)

**핵심 발견**:
- 이란: 사망률(4.67) 압도적 → 국가 권력의 강경 시위 탄압 패턴
- OPEC_Hormuz: 인구 노출도 최고 → 분쟁 시 에너지 공급망 위협 직결
- 03번(CAT 방식)과 보완 관계: 동일 데이터를 다른 축으로 분류하여 상호 검증

---

### `03_modeling_v4.ipynb`
**분위 회귀 & LightGBM Quantile — 모델 V1~V4 진화 과정**

GARCH-X로 전환하기 이전, 수준값 기반 분위 회귀와 트리 모델을 V1부터 V4까지 단계적으로 고도화하고 각 버전의 구조적 한계를 진단한다.

**노트북 목차**:

```
1. 데이터 로드
2. 피처 엔지니어링
3. EDA — 피처-타겟 상관관계
4. Train / Test Split (80/20, 시간 기준)
5. V1 — 기본 모델 (원본 10개 피처)
   └ 선형 분위 회귀 (QuantReg) + LightGBM Quantile
6. V1 → V2: 문제 진단 (VIF·분위 교차·시나리오 붕괴)
7. V2 — Conflict_Index 제거 (9개 피처)
8. V3 — VIF 기반 피처 축소 (5개 피처, 임계값=5)
9. V4 — 피처 복원 + Lag 효과 (13개 피처)
10. 시나리오 분석 (Best / Base / Worst)
11. 전체 버전 통합 비교 (Coverage / MAE / Pinball Loss / Winkler)
12. 예측 검증 — 2026-03-07 이후 실제 vs 예측
```

**버전별 핵심 변화 및 결과**:

| 버전 | 피처 수 | Coverage | MAE | 핵심 변화 |
|---|---|---|---|---|
| V1 | 10개 | 0.184 | 9.95 | 기본 모델. 분위 역전(Crossing) 0.544로 심각 |
| V2 | 9개 | 0.241 | 8.99 | Conflict_Index 제거 + Isotonic Sort 적용 |
| V3 | 5개 | — | — | VIF 임계값 5 기반 피처 제거. 유용 정보 손실 |
| V4 | 13개 | 0.609 | — | Lag 피처(1/5/20일) 추가. 목표치(0.80) 미달 |

**구조적 결론 — GARCH-X 전환 근거**:
- LightGBM은 Brent_lag1 자기상관을 항상 우선 학습 → GPR이 밴드 폭을 제어하는 구조 자체가 불가능
- 전쟁 국면처럼 학습 범위 밖의 GPR 수준에서는 외삽(extrapolation) 불가

---

### `04_modeling_v9.ipynb`
**GARCH-X Monte Carlo 최종 모델 (V9)**

GPR_THREAT를 분산 방정식에 직접 투입하는 GARCH-X 구조로 전환하고, V8의 3가지 한계를 개선하여 최종 모델(V9)을 완성한다. 시나리오별 1,000회 Monte Carlo 시뮬레이션으로 유가 변동 밴드를 도출한다.

**노트북 구성**:

```
1부: 방법론 선택 과정 — 시행착오 기록
  1.1 초기 설계: QuantReg + LightGBM → Brent_lag1 독점 문제
  1.2 두 번째 설계: Multi-horizon LightGBM → 이란전쟁 데이터 소실 문제
  1.3 GARCH-X로 전환하는 이유 (구조적 근거)

2부: GARCH-X 분석
  2.1 라이브러리 및 설정
  2.2 데이터 로드
  2.3 탐색적 분석: GARCH-X 적합성 근거
      ├ ADF 단위근 검정 (수준값 p=0.25, 수익률 p=0.00)
      ├ 변동성 군집 시각화 (절대수익률 ACF lag 1~10: +0.19~+0.27)
      └ GPR_THREAT vs 변동성 산점도 (δ=0.000081, p=0.0016)
  2.4 GARCH(1,1)-t vs EGARCH 비교 (AIC/BIC 기준)
  2.5 Monte Carlo 시뮬레이션: 미래 경로 생성
  2.6 핵심 시각화: 이벤트별 미래 경로 밴드 (D+0~D+60)
  2.7 GPR 레짐별 밴드 폭 비교 (설계 의도 검증)
  2.8 평시 vs 전쟁 국면 경로 직접 비교
  2.9 기업 대응 시나리오 (현재 시점 기준)
  2.10 모델 검증: 과거 이벤트 사후 적합성 확인 (Coverage / Escape Rate)
  2.11 롤링 팬 차트: D+0, D+5, D+10, D+21 시점 밴드 변화

3부: 분석 결과 요약
  3.1 방법론 선택 근거 요약
  3.5 종합 인사이트 및 시사점
  3.6 V9 업그레이드 사항 (V8 대비 3가지 개선)
```

**모델 구조**:

```
평균 방정식: r_t = μ + ε_t
분산 방정식: σ²_t = ω + α·ε²_(t-1) + β·σ²_(t-1) + δ·GPR_THREAT_(t-1)
오차항 분포: Hansen(1994) Skewed-t  (ν=6.37, λ=-0.033)
추정 방식:   Joint MLE — scipy.optimize.minimize (L-BFGS-B)
             대상 파라미터: μ, ω, α, β, δ, ν, λ (7개 동시 최적화)
GPR 감쇠:   GPR_t = 100 + (GPR_0 - 100) × 0.92^t  (AR(1), 장기평균 100 수렴)
시뮬레이션: 1,000회 × 42 영업일
```

**V8 대비 V9 핵심 개선**:

| # | 문제 | V8 방식 | V9 개선 |
|---|---|---|---|
| ① | δ 과소추정 | 2-Step OLS (GARCH 잔차에 회귀) | Joint MLE (7개 파라미터 동시 최적화) |
| ② | GPR 비현실적 발산 | 42일 내내 GPR 고정 | AR(1) 감쇠 → 장기 평균(100)으로 수렴 |
| ③ | 하방 과대 추정 | 대칭 t분포 (Q10이 -50%까지 하락) | Skewed-t λ=-0.033 (하방 경직성 반영) |

**주요 출력**:
- 4개 시나리오 × 이벤트별 유가 밴드 차트
- 파라미터 추정 결과표 (δ p-value 포함)
- Coverage Rate / Escape Rate 요약 (Early Phase D+1~10 vs Stable Phase D+11~42)

---

### `05_integrated_analysis.ipynb` ← **최종 통합본**
**전 분석 과정 통합 재현 노트북**

위 5개 노트북의 핵심 로직을 단일 파일에서 순서대로 재현한다. 성공한 분석뿐 아니라 실패·포기의 이유도 명시적으로 기록한다.

**목차 구성**:

```
Section 1.  환경 설정 — 전체 라이브러리 통합 임포트 및 경로 설정
Section 2.  데이터 전처리 — POPULATION_EXPOSURE 결측치 보완
            (M1 raw OLS → M2 log-y OLS → 최종 채택 과정 포함)
Section 3.  EDA — 지역 그룹 분류 (02_2 teammate 방식)
            GROUP_3 / GROUP_4 분류, 이란·OPEC_Hormuz 특성 분석
Section 4.  EDA — 이벤트 카테고리(CAT) 분류 (02_1 otto 방식)
            CAT1~4 건수, GPR 상관 구조 시각화
Section 5.  가설 검증 — 중동 데이터 (CAT·그룹별 히트맵)
Section 6.  가설 검증 — 글로벌 확장 + Granger 인과 (01 노트북 내용)
            중동 vs 글로벌 상관계수 급락 원인 분석
Section 7.  모델링 V1~V4 — 분위 회귀 & LightGBM Quantile (03 노트북 핵심)
            VIF 진단, 다중공선성 문제, 버전별 성능 비교
Section 8.  모델링 V5 — 다중 모델 앙상블 비교
            비정상성(ADF)·보간 누수·수렴 문제 3가지 발견
Section 9.  GARCH-X V8 — 방법론 전환 근거 + 2-Step OLS 한계
Section 10. GARCH-X V9 고도화 — Joint MLE + GPR 감쇠 + Skewed-t (04 노트북 핵심)
Section 11. 미해결 과제 및 향후 연구 방향 (failure.md 요약)
```

**이 노트북 하나로 확인할 수 있는 것**:
- 프로젝트 전체 분석 흐름 (EDA → 가설 → 모델링 → 결론)
- 각 단계에서 무엇을 시도했고 왜 실패했는지
- 최종 모델(GARCH-X V9)에 도달한 이유와 수치적 근거

---

## 노트북 간 의존 관계

```
01_가설검증_0330          02_1_eda_otto          02_2_eda_teammate
       │                        │                        │
       │                        └────────────┬───────────┘
       │                                     │
       └───────────── 03_modeling_v4 ────────┘
                             │
                      04_modeling_v9
                             │
              ╔══════════════╧══════════════╗
              ║   05_integrated_analysis    ║  ← 전 과정 통합본
              ╚═════════════════════════════╝

공통 입력: data/dataset_v28.csv
```


---

## 주요 커스텀 구현 목록

| 함수명 | 노트북 | 설명 |
|---|---|---|
| `neg_llk_skewt()` | `04`, `05` Section 10 | Hansen(1994) Skewed-t 로그우도 직접 구현 |
| `simulate_paths()` | `04`, `05` Section 10 | AR(1) GPR 감쇠 + Skewed-t 샘플링 Monte Carlo |
| `skewt_sample()` | `04`, `05` Section 10 | Skewed-t 분포 난수 샘플링 |
| `get_future_dates()` | `04`, `05` Section 10 | 이벤트 기준 영업일 날짜 시퀀스 생성 |
| `coverage_rate()` | `04`, `05` Section 10 | Early / Stable Phase별 Coverage Rate 계산 |
| `eval_summary()` | `03`, `05` Section 7 | Coverage / MAE / Pinball Loss / Winkler 통합 평가 |
