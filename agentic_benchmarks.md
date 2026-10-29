# Agent / Multi-Harness Routing Benchmarks

## Multi-Harness / Routing

| Name | Focus | GitHub |
|---|---|---|
| HM-ROUTER | Joint model + harness routing for agentic systems | https://github.com/hmarkc/HM-Router |
| Harness-Bench | Model-harness configuration evaluation across sandboxed agent tasks | https://github.com/Qihoo360/harness-bench |
| PawBench | LLM × harness co-evaluation benchmark | https://github.com/agentscope-ai/PawBench |
| LLMRouterBench | Model routing benchmark, not harness routing, but relevant baseline | https://github.com/ynulihao/LLMRouterBench |
| HAL Harness | Standardized evaluation harness for Holistic Agent Leaderboard | https://github.com/princeton-pli/hal-harness |
| Awesome Agent Harness | Survey and resource index for agent harnesses | https://github.com/Gloriaameng/Awesome-Agent-Harness |
| Awesome Agent Harness Benchmarks | Catalog of agent and harness benchmarks | https://github.com/zeredy879/awesome-agent-harness-benchmarks |

## Agent Benchmarks

| Name | Focus | GitHub |
|---|---|---|
| AgentBench | General LLM-as-agent benchmark across interactive environments | https://github.com/THUDM/AgentBench |
| General AgentBench | General-purpose agent evaluation across search, coding, reasoning, tool use | https://github.com/cxcscmu/General-AgentBench |
| MultiAgentBench / MARBLE | Multi-agent collaboration and competition benchmark | https://github.com/ulab-uiuc/MARBLE |
| MASBench | Multi-agent collaboration under partial observability; protocol, memory, routing | https://github.com/BUPT-GAMMA/MASBench |
| τ-bench | Tool-agent-user interaction benchmark | https://github.com/sierra-research/tau-bench |
| τ²-bench | Dual-control conversational agent benchmark | https://github.com/sierra-research/tau2-bench |
| Hyper-τ-bench | End-to-end realistic agent construction benchmark | https://github.com/sierra-research/hyper-tau-bench |
| ToolSandbox | Stateful conversational tool-use benchmark | https://github.com/apple/ToolSandbox |
| WebArena | Realistic web environment for autonomous agents | https://github.com/web-arena-x/webarena |
| OSWorld | Real desktop/computer-use agent benchmark | https://github.com/xlang-ai/OSWorld |
| SWE-bench | Real GitHub issue resolution benchmark | https://github.com/SWE-bench/SWE-bench |
| Terminal-Bench | Terminal/CLI agent benchmark | https://github.com/jvpoulos/terminal-bench |
| BrowseComp | Browsing/search agent benchmark | https://github.com/openai/simple-evals |

## Recommended Reading Order

1. HM-ROUTER
2. Harness-Bench
3. PawBench
4. HAL Harness
5. MASBench / MultiAgentBench
6. τ-bench / ToolSandbox
7. WebArena / OSWorld / Terminal-Bench / SWE-bench

## Quick Takeaway

- **HM-ROUTER** is the closest match for `multi-harness routing`.
- **Harness-Bench** is the most practical benchmark for measuring harness effects.
- **PawBench** and **HAL Harness** are useful for model × harness evaluation infrastructure.
- **MASBench** is relevant if the routing question is specifically about multi-agent collaboration mechanisms.
