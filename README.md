<img width="1024" height="559" alt="image" src="https://github.com/user-attachments/assets/9aa4e5c5-ccfc-4f8b-8fff-5c58a995c8af" /># stockontology

## MIROFISH-KG 온톨로지 추론 시뮬레이터 소스코드 명세서

본 소스코드는 시맨틱 웹 표준 기법인 OWL2-DL 구조의 온톨로지(TBox)와 지식그래프(ABox)를 결합하여, SWRL 규칙 기반의 전방향 연쇄 추론(Forward Chaining)을 웹 브라우저 상에서 시뮬레이션하고 검증할 수 있도록 구현된 단일 파일(Single-file) 웹 애플리케이션입니다.

GitHub 리포지토리의 `README.md` 또는 프로젝트 설명서로 바로 활용하실 수 있도록 핵심 설계와 모듈 구조를 개조식으로 요약·설명합니다.

---

<img width="1090" height="622" alt="image" src="https://github.com/user-attachments/assets/53b565f3-d5f0-4ad4-a3f6-b08effb3548d" />

<img width="1116" height="632" alt="image" src="https://github.com/user-attachments/assets/63b5a77a-1cf7-41c1-a727-d0757082f214" />

<img width="1042" height="629" alt="image" src="https://github.com/user-attachments/assets/93cc2ace-5515-408a-b58a-6e2d64ca4af9" />

## 1. 아키텍처 및 핵심 핵심 모듈 구조

소스코드는 외부 프레임워크(React, Vue 등) 없이 **Vanilla JS**와 D3.js (v7)를 활용하여 가볍고 독립적으로 동작하도록 설계되었습니다.

### ① 온톨로지 정의 (TBox) & 규칙 선언 (`ONTO`, `RULES`)

* **클래스 계층 구조(Class Hierarchy):** `MarketEntity`를 최상위 분기로 하여 강세 신호(`BullSignal`), 약세 신호(`BearSignal`), 촉매(`Catalyst`), 허브(`ClusterHub`), 가격 경로(`PricePath`) 등 도메인 개체들을 OWL 계층 구조 형태로 정의했습니다.
* **속성 정의(Properties):** 개체 간의 유기적 관계를 표현하는 객체 속성(`influences`, `amplifies`, `contradicts`, `aggregatedBy`, `precedes`)과 리터럴 값을 갖는 데이터 속성(`hasWeight`, `hasConfidence`)을 설정했습니다.
* **SWRL 추론 규칙:** `R1`부터 `R10`까지 총 10개의 전방향 추론 규칙을 선언했습니다. 특정 임계치(0.62) 이상의 가중치를 가진 신호를 분류하고, 촉매 결합 시 가중치 상향(+0.06), 상반 신호 대치 시 모순 감쇠(가중치 $\times 0.5$) 메커니즘을 수학적으로 반영합니다.

### ② 추론 엔진 (`infer()`)

* **전방향 연쇄(Forward Chaining) 구현:** 초기 지식그래프 데이터에 선언된 규칙(`RULES`)을 순차적으로 적용하여 고정점(Fixpoint)에 도달할 때까지 새로운 트리플(Inferred Facts)을 유도합니다.
* **확률론적 귀결:** 최종적으로 누적된 상승 경로 기여도($U$)와 하락 경로 기여도($D$)를 집계하여 시장의 최종 도출 방향 확률 $P(\text{UP}) = \frac{U}{U+D}$를 계산합니다.
* **추론 트레이스 기록:** 모든 규칙의 발화(Firing) 순간을 `TRACE` 배열에 추적·기록하여 UI 상에서 시각적 애니메이션으로 재현할 수 있는 기반을 제공합니다.

### ③ 지식그래프 시각화 모듈 (D3.js Force-Directed Graph)

* **힘-지향 그래프(Force-Directed Layout):** D3.js의 충돌 방지(`forceCollide`), 전하력(`forceManyBody`), 중심력(`forceCenter`) API를 사용하여 노드와 링크를 브라우저 스페이스에 동적으로 배치합니다.
* **허브 앵커링(Anchoring):** 그래프의 인지 편의성을 위해 주요 클러스터 허브(`HUB_PRIME`, `BEAR_CLUSTER` 등)의 좌표($fx$, $fy$)를 고정 레이아웃으로 바인딩했습니다.
* **인터랙티브 하이라이트:** 사용자가 특정 노드를 클릭하거나 추론 단계를 진행할 때 인접 트리플 및 추론 유도 경로에 불빛이 들어오거나 대시 스트로크 애니메이션(`flowdash`)이 작동하도록 구현했습니다.

### ④ 재현성 검증 시뮬레이터 (Monte Carlo & PRNG)

* **Mulberry32 PRNG:** 자바스크립트 기본 `Math.random()`의 시드 고정 불가 문제를 해결하기 위해 시드 기반 32비트 의사난수 생성기를 내장했습니다.
* **FNV-1a 64비트 해시 알고리즘:** 시뮬레이션 결과 배열을 비트 연산 기반 해시 코드로 변환하여 "동일 시드 = 동일 해시 결과"라는 결정론적 재현성(Reproducibility 100%)을 검증합니다.
* **몬테카를로 분석:** 60회 이상의 교차 시드 분산 분석을 수행하고 대수의 법칙에 따른 수렴도 그래프(`ch-conv`) 및 Win-Rate 분포 히스토그램(`ch-hist`)을 실시간 렌더링합니다.

### ⑤ SPARQL 질의응답 및 채점 엔진 (`scoreQA()`)

* **가상 SPARQL 벤치마크:** 가상의 SPARQL 쿼리 문장 인터페이스를 제공하고, 내부 룰 엔진이 가상 그래프를 직접 탐색(`run()`)하여 정답셋(`gold`)과 교집합을 비교합니다.
* **정량적 스코어링:** 시스템의 응답 정확도(60점), 근거 충실도(25점), 추론 깊이(15점)를 합산하여 총 100점 만점 기준으로 다차원 벤치마크 점수를 시각화합니다.

---

## 2. 소스코드의 강점 (GitHub 어필 포인트)

1. **No-Dependency Web App:** 외부 무거운 백엔드 서버(Fuseki, Blazegraph 등)나 추론기(Pellet, HermiT) 없이 클라이언트 브라우저 사이드 단독으로 시맨틱 추론 풀스택을 에뮬레이션합니다.
2. **Deterministic Simulation:** 정밀한 시드 고정 알고리즘을 도입하여 난수 기반 몬테카를로 시뮬레이션 환경에서도 완벽한 디버깅 및 재현 검증이 가능합니다.
3. **Semantic Web and Data Science Fusion:** 온톨로지를 통한 상징적 인공지능(Symbolic AI) 기법과 통계적 시뮬레이션 기법을 융합한 훌륭한 레퍼런스 아키텍처입니다.

궁금한 부분이나 추가적으로 백엔드 연동 및 리팩토링이 필요한 부분이 있으시면 언제든 말씀해 주세요, 브라이언 박사과정님!
