# Agent / Multi-Harness Routing Benchmarks 정리

기준일: 2026-10-29

> 주의: Agent benchmark 점수는 대부분 `model × harness × tool/runtime × evaluator × version` 조합의 결과다.  
> 따라서 서로 다른 benchmark의 점수를 직접 비교하면 안 되고, 같은 benchmark 안에서도 version, harness, reasoning effort, tool setting을 함께 봐야 한다.

## Quick Recommendation

| 목적 | 먼저 볼 벤치마크 |
|---|---|
| Multi-harness routing | HM-ROUTER |
| Harness별 성능 차이 | PawBench, Harness-Bench |
| Agent benchmark 통합 실행 / 비용 추적 | HAL |
| Multi-agent routing / coordination | MASBench, MultiAgentBench |
| Tool-use 안정성 | τ-bench, ToolSandbox |
| Web agent | WebArena, BrowseComp |
| Desktop / computer-use agent | OSWorld |
| Coding agent | SWE-bench |
| Terminal agent | Terminal-Bench |

## Benchmark Overview

| Benchmark | GitHub / Link | 도메인 / 카테고리 | 규모 | 주요 리더보드 / 스코어 | 측정 방식 | 필요 리소스 | 무엇을 측정하나 |
|---|---|---|---:|---|---|---|---|
| HM-ROUTER | https://github.com/hmarkc/HM-Router | Agent benchmark 전반, model-routing, harness-routing | 293 routes, 73 models, 25 harnesses | strongest learned baseline 대비 mean routing accuracy +7.3pp | query별 최적 `model × harness` 선택 정확도 및 cost-budget별 성능 | route outcome data, 모델 API, harness 실행환경 | query마다 어떤 모델과 harness 조합을 써야 하는지 |
| Harness-Bench | https://github.com/Qihoo360/harness-bench | Software engineering, business ops, systems, research, web dev, content production | 300 sandboxed tasks, 7 harnesses, 10 model backends, 1,408 scored modules | 안정적 public leaderboard보다는 benchmark framework 중심 | hidden oracle이 final artifact/workspace state를 scoring. safety gate 포함 | Docker, uv/Python, harness images, model endpoint, task archive | 같은 모델이 harness에 따라 얼마나 달라지는지 |
| PawBench | https://github.com/agentscope-ai/PawBench | Production-grade agent tasks, text + multimodal | 150 tasks, 3 harnesses, 9 models | 1위 `claude-opus-4.6 + Hermes`: 78.4 / 2위 `qwen3.6-max-preview + QwenPaw`: 78.3 | Python grader + LLM judge 혼합 | Docker, model endpoint, QwenPaw/OpenClaw/Hermes | `model × harness` 조합의 실제 agent 성능 |
| LLMRouterBench | https://github.com/ynulihao/LLMRouterBench | Model routing, performance-cost routing | 400K+ instances, 21 datasets, 33 models, 10 routers | 고정 leaderboard보다는 routing method 비교 | query별 모델 선택 성능, cost-saving, Pareto trade-off | precomputed outputs, router model, embeddings, optional inference API | 어떤 query를 어떤 모델에 보낼지 |
| HAL / Holistic Agent Leaderboard | https://github.com/princeton-pli/hal-harness | GAIA, SWE-bench, τ-bench, web/coding/science tasks 등 | 9 benchmarks, 수만 rollouts | 예: GAIA 1위 Claude Sonnet 4.5 74.5%, SWE-bench Verified Mini 72.0% | benchmark별 native evaluator + cost/token/trace logging | Docker/local/Azure VM, model API, Weave logging | 여러 agent benchmark를 표준화하고 비용까지 추적 |
| AgentBench / AgentBench-FC | https://github.com/THUDM/AgentBench | OS, DB, KG, ALFWorld, WebShop 등 | original 8 envs, FC는 5 envs | AgentBench-FC 1위 `AgentRL w/ Qwen2.5-32B`: 70.4 avg pass@1 | 환경별 interactive success / pass@1 | Docker, MySQL, Ubuntu, task workers, model API | LLM이 interactive environment에서 agent처럼 행동하는 능력 |
| General AgentBench | https://github.com/cxcscmu/General-AgentBench | Search, coding, reasoning, tool-use | 여러 benchmark를 MCP로 통합 | 별도 live leaderboard보다는 논문 실험 중심 | native evaluator + parallel/sequential scaling | MCP servers, Docker, Serper API, model API | 하나의 general agent가 여러 도메인을 함께 처리하는지 |
| MultiAgentBench / MARBLE | https://github.com/ulab-uiuc/MARBLE | Research, Minecraft, DB, coding, bargaining, Werewolf | 6 interactive scenarios | `gpt-4o-mini`가 평균 task score 최고. Research TS 84.13%, Coding TS 65.10 | Task Score, Coordination Score, milestone KPI | MARBLE framework, multi-agent role setup, model API | multi-agent collaboration/competition과 coordination 품질 |
| MASBench | https://github.com/BUPT-GAMMA/MASBench | Reasoning, Scheduling, Game | 공개 논문/코드 중심 | 아직 안정적 public leaderboard보다는 benchmark 중심 | performance score, communication cost, cost effectiveness | LLM API, multi-agent orchestration, category별 task data | 부분 관측 환경에서 protocol/memory/routing의 효과 |
| τ-bench / τ³-Banking | https://github.com/sierra-research/tau-bench | Retail, airline, banking, telecom customer service | 원본 165 tasks. τ³-Banking은 약 700문서 KB | τ³-Banking 1위 Qwen 3.8 Max 55.2% pass^1, Claude Opus 5 48.7% | final DB state correctness, pass^k | DB/API simulator, user simulator LLM, policy docs, model API | tool-using customer-service agent의 안정적 task completion |
| ToolSandbox | https://github.com/apple/ToolSandbox | Stateful conversational tool use | 1,032 test cases, 34 tools | 논문 결과 GPT-4o avg similarity 73.0, Claude-3-Opus 69.2 | Milestones/Minefields DAG matching | tool execution env, user simulator, model API | state dependency, clarification, hallucinated tool calls, tool robustness |
| WebArena | https://github.com/web-arena-x/webarena | Shopping, admin, GitLab, forum, wiki, map | 812 tasks | Original leaderboard: Narada AI Code 86.1%, DeepSeek v3.2 74.3%, human baseline 78.24% | website final state, URL/string/programmatic checks | self-hosted websites, browser automation, evaluator | browser agent가 실제 웹 task를 수행하는 능력 |
| OSWorld | https://github.com/xlang-ai/OSWorld | Desktop GUI, browser, office apps, terminal, multi-app workflow | v1: 369 tasks, v2.0: 108 long-horizon workflows | OSWorld 2.0 binary: Claude Opus 5 48.65%, GPT-5.6 Sol 28.1% | execution-based final state / partial score | VM/Docker+KVM, GUI automation, screenshots, model API | real computer-use agent 능력 |
| SWE-bench | https://github.com/SWE-bench/SWE-bench | Software engineering, GitHub issue repair | 원본 2,294 issues, Verified 500 | SWE-bench Verified 1위권 79.20% resolved | fail-to-pass / pass-to-pass tests | Docker, repo checkout, test deps, coding harness | 실제 GitHub issue를 patch로 해결하는 능력 |
| Terminal-Bench | https://github.com/jvpoulos/terminal-bench | CLI, infra, coding, data/science workflows | v2.0: 89 hard tasks, v4.0 별도 | Terminal-Bench 4.0 snapshot: GPT-6 Astra + Codex 58.2% ± 2.8% | containerized terminal task resolution rate | Docker/Harbor, task images, verifier, model API | terminal 환경에서 long-horizon task 수행 능력 |
| BrowseComp | https://github.com/openai/simple-evals | Hard web browsing, multi-hop factual search | 1,266 questions | OpenAI 원 결과: Deep Research 51.5%, o1 9.9%, GPT-4o browsing 1.9% | short answer exact/semantic verification | browsing/search tool, internet access, evaluator, model API | 찾기 어려운 정보를 끈질기게 탐색하는 능력 |
| Awesome-Agent-Harness | https://github.com/Gloriaameng/Awesome-Agent-Harness | Harness survey / catalog | 168 references, 34 systems, 30 benchmarks | 점수 없음 | benchmark가 아니라 index | 없음 | harness/benchmark 후보 탐색 |
| Awesome-Agent-Harness-Benchmarks | https://github.com/zeredy879/awesome-agent-harness-benchmarks | Benchmark catalog | catalog 성격 | 점수 없음 | benchmark가 아니라 index | 없음 | agent/harness benchmark 목록 정리 |

## 핵심 해석

### 1. Multi-Harness Routing에 가장 직접적인 것

**HM-ROUTER**가 가장 직접적이다.

기존 model router는 보통 query를 보고 어떤 모델을 쓸지 고른다.  
HM-ROUTER는 한 단계 더 나아가 query마다 어떤 **model + harness pair**를 써야 하는지 고른다.

즉, 질문이 다음과 같다면 HM-ROUTER가 맞다.

> “이 task는 Claude를 Claude Code에 태우는 게 나은가, GPT를 Codex harness에 태우는 게 나은가?”

### 2. Harness 효과를 보려면

**PawBench**와 **Harness-Bench**가 중요하다.

- PawBench는 공개 score matrix가 깔끔해서 바로 비교하기 좋다.
- Harness-Bench는 더 큰 task taxonomy와 hidden oracle 기반 grading을 제공한다.
- 둘 다 “모델만 좋으면 되는가?”가 아니라 “모델을 어떤 harness에 태우느냐가 성능을 바꾼다”를 보여준다.

### 3. Multi-Agent Routing / Coordination을 보려면

**MASBench**와 **MultiAgentBench**를 봐야 한다.

- MASBench는 protocol, memory, routing을 명시적으로 다룬다.
- MultiAgentBench는 collaboration/competition, coordination topology, planning strategy를 평가한다.

### 4. Tool-use 안정성을 보려면

**τ-bench**와 **ToolSandbox**가 좋다.

- τ-bench는 customer-service workflow에서 DB/API/policy를 잘 따르는지 본다.
- ToolSandbox는 state dependency, insufficient information, hallucinated tool call 같은 failure mode를 더 촘촘하게 본다.

### 5. 실전형 agent 능력을 보려면

| 능력 | Benchmark |
|---|---|
| 웹 조작 | WebArena |
| 웹 리서치 | BrowseComp |
| 데스크톱 조작 | OSWorld |
| 실제 코드 수정 | SWE-bench |
| 터미널 작업 | Terminal-Bench |

## 점수 해석 시 주의사항

1. **같은 benchmark라도 version이 다르면 비교 불가**
   - 예: Terminal-Bench 1.0, 2.x, 4.0은 난이도와 task set이 다르다.
   - OSWorld v1, OSWorld-Verified, OSWorld 2.0, 2.1도 구분해야 한다.

2. **agent score는 모델 점수가 아니다**
   - `Claude + Claude Code`
   - `Claude + OpenHands`
   - `GPT + Codex`
   - `GPT + custom harness`

   위 조합은 같은 모델이어도 전혀 다른 점수를 낼 수 있다.

3. **self-reported leaderboard는 조심해서 봐야 한다**
   - WebArena original leaderboard처럼 self-reported 성격이 강한 경우가 있다.
   - trajectory, evaluator, run count, harness commit이 공개되어 있는지 확인해야 한다.

4. **cost와 latency를 같이 봐야 한다**
   - HAL, LLMRouterBench, HM-ROUTER 계열은 성능뿐 아니라 cost-performance trade-off가 중요하다.
   - agent benchmark는 multi-turn이라 API 비용이 빠르게 커진다.

5. **pass@1과 pass^k는 다르다**
   - pass@1: 한 번 시도해서 성공했는가
   - pass^k: k번 반복해도 안정적으로 성공하는가
   - τ-bench는 반복 안정성을 중요하게 본다.

## 연구/실험용 우선순위

### Multi-Harness Routing 연구

1. HM-ROUTER
2. LLMRouterBench
3. PawBench
4. Harness-Bench
5. HAL

### Harness 성능 비교

1. PawBench
2. Harness-Bench
3. HAL
4. Terminal-Bench
5. SWE-bench

### Multi-Agent 연구

1. MASBench
2. MultiAgentBench / MARBLE
3. τ-bench
4. ToolSandbox

### 실제 제품/agent 평가

1. SWE-bench
2. Terminal-Bench
3. OSWorld
4. WebArena
5. τ-bench
6. BrowseComp
