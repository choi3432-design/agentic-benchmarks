# Budget + User Preference 기반 LLM / Agent Routing 논문 정리

기준일: 2026-10-29

## 핵심 결론

“한정된 budget 안에서 사용자 선호까지 반영해서 routing”하는 연구는 존재한다.

현재 흐름은 크게 세 가지다.

1. **Personalized / preference-aware routing**
   - 사용자별 cost-performance-style 선호를 학습해 모델을 선택

2. **Budget / SLA constrained routing**
   - 비용 제한, SLA, 만족도 보장 조건 아래에서 routing

3. **Agent / multi-turn routing으로의 확장**
   - 대화 이력, agent trajectory, step-level routing까지 고려

아직 `user preference + hard budget + multi-turn agent/harness routing`을 모두 강하게 결합한 대표 논문은 적다.  
그래서 이 세 방향을 결합하는 것이 좋은 연구 빈틈으로 보인다.

---

## 1. 핵심 논문 요약

| 분류 | 논문 | 링크 | 핵심 아이디어 | 관련성 |
|---|---|---|---|---|
| Personalized + Budget + Online | Chance-Constrained Personalized LLM Routing with Online Preference Learning | https://arxiv.org/abs/2610.22695 | 사용자 선호를 online feedback으로 학습하면서 cumulative satisfaction constraint를 만족하도록 cost 최소화 | 가장 직접적인 `budget + user preference + online routing` |
| Personalized + Meta-learning | Learning to Route LLMs from Implicit Cost-Performance Preferences via Meta-Learning / MetaRouter | https://arxiv.org/abs/2606.06178 | 사용자의 암묵적 cost-performance preference를 few interaction으로 추론하고 routing | 명시적 budget slider 없이 사용자 선호 학습 |
| User-Controlled Quality-Cost | IPR: Intelligent Prompt Routing with User-Controlled Quality-Cost Trade-offs | https://aclanthology.org/2025.emnlp-industry.170.pdf | 사용자가 tolerance `τ`를 지정하면 품질 허용치 내에서 가장 싼 모델 선택 | 실서비스형 user-controlled routing |
| SLA / Satisfaction Guarantee | SLARouter: Cost-Optimal LLM Routing with Limited User Feedback under User Satisfaction Guarantees | https://arxiv.org/abs/2606.19376 | sparse one-sided feedback만으로 SLA 만족을 보장하면서 cost-optimal routing | 사용자 선호를 satisfaction/SLA로 모델링 |
| Personalized Graph Router | PersonalizedRouter | https://arxiv.org/abs/2511.16883 | user-query-LLM interaction을 heterogeneous graph로 만들고 개인별 LLM 선택 학습 | cost, performance, response style 선호 반영 |
| Multi-turn Personalized Router | GMTRouter | https://arxiv.org/abs/2511.08590 | user, LLM, query, response를 graph node로 두고 multi-turn interaction history 기반 routing | 대화 이력을 통한 사용자 preference 추론 |
| Preference-aware Evaluation | RouteJudge | https://arxiv.org/abs/2606.18774 | budget constraint 아래 router 결과를 anonymous pairwise preference로 평가 | preference-aware router 평가 플랫폼 |
| Routing Infra / Benchmark | LLMRouter / xRouteBench | https://arxiv.org/abs/2608.06867 | single-turn, multi-turn, personalized routing을 unified infra로 비교 | personalized routing과 cost-aware evaluation 포함 |

---

## 2. Chance-Constrained Personalized LLM Routing with Online Preference Learning

- 링크: https://arxiv.org/abs/2610.22695
- 날짜: 2026-10
- 키워드:
  - personalized LLM routing
  - online preference learning
  - constrained contextual bandit
  - chance constraint
  - cumulative satisfaction guarantee

### 문제 설정

사용자마다 같은 query에 대해 선호하는 모델이 다를 수 있다.

```text
same query
different users
different preferred models
```

router는 다음을 동시에 해야 한다.

```text
1. 사용자 선호를 online feedback으로 학습
2. cumulative satisfaction을 일정 확률 이상 보장
3. inference cost 최소화
```

논문에서는 satisfaction constraint를 다음처럼 둔다.

```text
P(sum_t Y_t >= (1 - epsilon) T) >= 1 - delta
```

즉 전체 T번 요청 중 최소 `(1 - epsilon)` 비율 이상은 만족스러운 응답이어야 하고, 이 조건이 높은 확률로 보장되어야 한다.

### 방법론

- 문제를 **constrained contextual bandit**으로 formulate
- 모델마다 cost와 satisfaction probability가 다름
- 사용자 선호는 unknown parameter로 두고 online feedback으로 학습
- router는 cost를 줄이되 satisfaction deficit이 커지면 더 안전한 모델 선택
- optimistic estimate를 사용해 exploration과 exploitation 균형

### 왜 중요?

단순 평균 만족도가 아니라, 실제 deployment에서 **실현된 누적 만족도**가 기준 아래로 떨어지지 않도록 제어한다.  
“비용을 아끼되 사용자가 불만족하는 응답이 너무 많이 나오면 안 된다”는 setting에 가장 가깝다.

---

## 3. MetaRouter: Learning to Route LLMs from Implicit Cost-Performance Preferences via Meta-Learning

- 링크: https://arxiv.org/abs/2606.06178
- IJCAI 2026
- 키워드:
  - perceptive LLM routing
  - meta-learning
  - contextual bandit
  - implicit preference
  - cost-performance trade-off

### 문제 설정

기존 router는 사람이 직접 trade-off parameter를 정한다.

```text
performance_weight = 0.8
cost_weight = 0.2
```

하지만 실제 사용자는 이런 숫자를 잘 모른다.  
MetaRouter는 사용자에게 적은 수의 pairwise comparison을 받고, 사용자의 latent preference를 추론한다.

```text
query + user preference context -> routing policy
```

### 방법론

- preference profile을 서로 다른 task로 봄
- 다양한 preference profile에서 meta-training
- user adaptation 단계에서 pairwise comparison 수집
- context encoder가 latent preference representation 추론
- routing policy는 query와 preference representation을 함께 사용
- contextual bandit으로 routing decision 학습

### 왜 중요?

사용자가 직접 “나는 cost를 0.3만큼 중요하게 봐”라고 말하지 않아도, interaction을 통해 사용자의 암묵적 선호를 배운다.

---

## 4. IPR: Intelligent Prompt Routing with User-Controlled Quality-Cost Trade-offs

- 링크: https://aclanthology.org/2025.emnlp-industry.170.pdf
- EMNLP Industry 2025
- AWS AI / Amazon Bedrock 계열
- 키워드:
  - user-controlled quality-cost trade-off
  - tolerance parameter
  - quality estimator
  - production routing

### 문제 설정

사용자가 quality tolerance `τ`를 정하면 router가 그 허용치 내에서 가장 비용 효율적인 모델을 고른다.

```text
τ = 0: strict quality
τ = 1: lenient quality
```

### 방법론

- 1.5M prompt-response quality annotation
- 11개 LLM 후보
- lightweight quality estimator
- frozen encoder + model-specific adapter
- 각 candidate model의 predicted quality를 추정
- tolerance를 만족하는 후보 중 가장 싼 모델 선택

### 주요 결과

- strongest Claude family model과 quality parity 유지
- cost 43.9% reduction
- low latency production routing

### 왜 중요?

연구용보다는 실제 제품형 routing에 가깝다.  
사용자가 quality-cost trade-off를 직접 제어할 수 있다는 점이 강점이다.

---

## 5. SLARouter: Cost-Optimal LLM Routing with Limited User Feedback under User Satisfaction Guarantees

- 링크: https://arxiv.org/abs/2606.19376
- 날짜: 2026-06
- 키워드:
  - SLA guarantee
  - sparse user feedback
  - one-sided observational feedback
  - online routing
  - cost optimality

### 문제 설정

실서비스에서는 대부분 request에 explicit feedback이 없다.

```text
feedback rate < 5%
```

그리고 feedback이 있더라도 선택된 모델 응답에 대한 one-sided feedback만 있다.

SLARouter는 이런 제한된 feedback으로도 SLA constraint를 만족하면서 cost를 줄이는 router를 학습한다.

### 방법론

- sparse, one-sided observational feedback 사용
- online routing algorithm
- user satisfaction을 SLA constraint로 모델링
- cost-optimal policy를 학습
- per-benchmark tuning 없이 strict SLA compliance 목표

### 주요 결과

- SLA constraint 만족
- baseline 대비 operating cost 최대 2.2x 절감

### 왜 중요?

개별 취향보다는 “사용자 만족도를 일정 수준 이상 보장”하는 production setting에 적합하다.

---

## 6. PersonalizedRouter

- 링크: https://arxiv.org/abs/2511.16883
- GitHub: https://github.com/ulab-uiuc/PersonalizedRouter
- 키워드:
  - personalized LLM selection
  - graph-based user preference modeling
  - PersonaRoute-Bench
  - response style preference

### 문제 설정

사용자는 performance, cost, response style에 대해 서로 다른 선호를 가진다.

PersonalizedRouter는 interaction data를 heterogeneous graph로 변환한다.

```text
node types:
- user
- query
- task context
- candidate LLM
- decision / preference
```

### 규모

- PersonaRoute-Bench
- 1,000 simulated users
- 10 LLMs

### 방법론

- user-query-LLM interaction을 heterogeneous graph로 구성
- graph neural network / graph-based representation learning
- user별 preference representation 학습
- candidate LLM의 suitability 예측
- new user / new LLM few-shot generalization 평가

### 주요 결과

- 기존 LLM selection method보다 큰 폭으로 성능 향상
- new user / new LLM에 대한 few-shot generalization 보고

### 왜 중요?

“사용자별 모델 선택”을 직접적으로 다룬다.  
다만 hard budget constraint를 enforce한다기보다는 cost preference를 user preference의 한 축으로 다룬다.

---

## 7. GMTRouter

- 링크: https://arxiv.org/abs/2511.08590
- GitHub: https://github.com/ulab-uiuc/GMTRouter
- 키워드:
  - personalized LLM router
  - multi-turn interaction
  - heterogeneous graph
  - few-shot personalization

### 문제 설정

개인화 routing에서 중요한 정보는 단일 query가 아니라 대화 이력이다.

GMTRouter는 multi-turn user-LLM interaction을 graph로 표현한다.

```text
node types:
- user
- LLM
- query
- response
```

### 방법론

- multi-turn interaction history를 heterogeneous graph로 구성
- tailored message passing으로 user preference 추론
- few-shot user data로 personalization
- new user / evolving preference에 adaptation

### 주요 결과

- strong baseline 대비 accuracy와 AUC 개선
- few-shot data만으로 new user와 evolving preference에 적응

### 왜 중요?

실제 assistant나 agent에서는 user preference가 대화 이력에 묻혀 있다.  
GMTRouter는 이 multi-turn personalization 문제에 직접 가깝다.

---

## 8. RouteJudge

- 링크: https://arxiv.org/abs/2606.18774
- 프로젝트: https://lamda-model-reuse.github.io/RouteJudge/
- 키워드:
  - preference-aware router evaluation
  - pairwise comparison
  - budget-aware evaluation
  - ORBIT toolbox

### 문제 설정

LLM router 평가는 모델 자체 평가와 다르다.  
좋은 router는 같은 budget 아래에서 더 사용자가 선호하는 응답을 내는 모델을 고른다.

RouteJudge는 여러 routing strategy가 고른 model response를 anonymous pairwise comparison으로 사용자에게 보여주고, preference label을 router strategy에 attribution한다.

### 저장하는 정보

```text
query
routing decision
model response
preference label
cost
latency
task metadata
```

### 왜 중요?

사용자 선호 기반 router를 비교하려면 이런 평가 플랫폼이 필요하다.

---

## 9. LLMRouter / xRouteBench

- 링크: https://arxiv.org/abs/2608.06867
- 프로젝트: https://ulab-uiuc.github.io/LLMRouter/
- 키워드:
  - unified routing infrastructure
  - single-turn routing
  - multi-turn routing
  - personalized routing
  - cost-aware evaluation

### 방법론

LLM routing을 다섯 요소로 통합한다.

```text
1. context encoders
2. model encoders
3. scoring functions
4. decision rules
5. learning signals
```

### 범위

xRouteBench는 다음을 포함한다.

```text
generic LLM routing
memory-augmented routing
vision routing
time-series routing
personalized routing
```

### 주요 결과

- learned router가 strongest fixed-model baseline보다 14.6% relative improvement
- tight cost constraint에서는 lightweight router가 더 경쟁력 있음
- user-conditioned routing이 personalization에서 일관되게 유리

### 왜 중요?

개별 논문보다 routing 방법들을 같은 프레임으로 비교하기 좋다.  
personalized routing과 cost-aware evaluation을 같이 보기 위한 기반으로 유용하다.

---

## 10. Budget 중심 인접 논문

아래 논문들은 user preference 자체보다는 **budget constraint**가 중심이지만, 사용자 선호 routing과 결합하기 좋다.

| 논문 | 링크 | 핵심 |
|---|---|---|
| WISERouter | https://arxiv.org/abs/2607.23765 | workload-level budget constraint를 constrained contextual MAB로 다룸 |
| PILOT / Adaptive LLM Routing Under Budget Constraints | https://aclanthology.org/2025.findings-emnlp.1301.pdf | preference-prior informed LinUCB + multi-choice knapsack |
| R2-Router | https://proceedings.mlr.press/v306/xue26h.html | model뿐 아니라 output length budget도 같이 선택 |
| RouteLLM | https://proceedings.iclr.cc/paper_files/paper/2025/hash/5503a7c69d48a2f86fc00b3dc09de686-Abstract-Conference.html | preference data로 strong/weak model routing 학습 |
| MESS+ | https://papers.neurips.cc/paper_files/paper/2025/hash/4dd1d9b841712bd37b833559f041530c-Abstract-Conference.html | cost-optimal routing + SLA compliance |
| PROTEUS | https://arxiv.org/abs/2601.19402 | Lagrangian RL로 runtime accuracy target을 만족하는 routing |

---

## 11. Agent / Multi-Turn 확장에 가까운 논문

아래 논문들은 user preference보다는 **multi-turn agent routing / step-level cost optimization**이 중심이다.  
하지만 preference-aware routing과 결합하기 좋은 재료다.

| 논문 | 링크 | 핵심 |
|---|---|---|
| MTRouter | https://aclanthology.org/2026.acl-long.2045/ | history-model joint embedding으로 turn-level model utility 예측 |
| RIVET | https://arxiv.org/abs/2610.23239 | response-informed value estimation으로 delayed reward 문제 완화 |
| Whose Turn Is It? | https://arxiv.org/abs/2610.19292 | stateful agent routing에서 session history와 prompt-cache switching cost 고려 |
| AgentRouter | https://arxiv.org/abs/2609.22951 | agent trajectory step별 model tier 선택 |
| CarbonRouter | https://arxiv.org/abs/2610.35230 | multi-turn session의 KV-cache와 carbon-aware deployment routing |

---

## 12. 연구 방향 제안

### A. User Preference + Agent Multi-Turn Routing 결합

현재 논문들은 대체로 둘 중 하나에 집중한다.

```text
PersonalizedRouter / GMTRouter:
사용자 선호 modeling은 강하지만 agent trajectory routing은 약함

MTRouter / RIVET / AgentRouter:
agent trajectory routing은 강하지만 사용자 선호 modeling은 약함
```

좋은 연구 질문:

```text
Can we route each step or turn of an agent trajectory under a limited budget,
while adapting to user-specific preferences over quality, cost, latency, style, privacy, and risk?
```

---

### B. Preference를 다차원 utility로 보기

사용자 선호는 단순 정답률이 아니다.

```text
utility(user, query, route)
= w_quality * quality
- w_cost * cost
- w_latency * latency
- w_risk * risk
- w_privacy * privacy_cost
+ w_style * style_match
+ w_explainability * explanation_score
```

고려할 축:

- accuracy
- cost
- latency
- verbosity
- response style
- privacy
- risk tolerance
- tool-use aggressiveness
- local vs cloud preference
- determinism
- explainability

---

### C. Harness까지 포함한 확장

HM-ROUTER는 `model × harness` routing을 다룬다.  
여기에 user preference와 budget을 붙이면 다음 문제가 된다.

```text
select:
  model m
  harness h
  tool policy p
  reasoning budget b
  output budget l

subject to:
  workload budget
  latency constraint
  user satisfaction constraint
  safety constraint
  privacy constraint
```

이 조합은 아직 대표 논문이 적어서 연구 가치가 크다.

---

## 13. 추천 읽는 순서

1. RouteLLM
2. IPR
3. MetaRouter
4. Chance-Constrained Personalized Routing
5. SLARouter
6. PersonalizedRouter
7. GMTRouter
8. RouteJudge
9. LLMRouter / xRouteBench
10. WISERouter
11. MTRouter
12. RIVET
13. AgentRouter
14. HM-ROUTER

---

## 14. 한 줄 결론

현재 최신 연구는 **budget-aware routing**, **preference-aware personalized routing**, **agent/multi-turn routing**으로 나뉘어 발전 중이다.

가장 유망한 연구 빈틈은 다음이다.

```text
사용자별 선호와 제한된 예산을 고려해,
agent trajectory의 각 step마다
model, harness, reasoning budget, tool policy를 선택하는 router
```
