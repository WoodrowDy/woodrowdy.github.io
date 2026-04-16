---
title: "Context Engineering & Harness Engineering: AI 에이전트 시대의 두 핵심 엔지니어링"
date: 2026-04-13 11:21:00 +0900
categories: [AI Engineering, Agent]
tags: [context-engineering, harness-engineering, llm, agent, nestjs]
---

프롬프트 하나 잘 쓰면 되던 시대는 끝났다. 2026년, AI 에이전트를 프로덕션에 올리려면 "컨텍스트"와 "하네스"를 설계해야 한다. 백엔드 개발자 관점에서 두 분야의 이론을 정리했다.

---

## 들어가며

2025년은 AI 에이전트가 "코드를 쓸 수 있다"는 것을 증명한 해였다. 2026년은 "에이전트가 아니라, 에이전트를 둘러싼 시스템이 어려운 부분"이라는 것을 깨달은 해다.

OpenAI의 Codex 팀은 엔지니어 3명이 에이전트를 활용해 100만 줄 이상의 코드베이스를 구축했다[^1]. 5개월간 약 1,500개의 PR을 머지하며 엔지니어당 하루 3.5개 PR이라는 처리량을 기록했다. 핵심은 모델의 능력이 아니라 모델을 둘러싼 인프라의 설계였다.

이 글에서는 이 인프라를 설계하는 두 가지 엔지니어링 분야 — **Context Engineering**과 **Harness Engineering** — 의 이론을 정리한다.

---

## 1. Context Engineering (컨텍스트 엔지니어링)

### 1.1 정의

Context Engineering은 LLM에게 전달되는 전체 입력 환경을 체계적으로 설계하는 분야다. 프롬프트 엔지니어링이 "좋은 질문 하나를 쓰는 기술"이었다면, 컨텍스트 엔지니어링은 "모델이 올바른 판단을 내리기 위해 필요한 모든 정보를 최적의 형태로 구성하는 기술"이다.

Anthropic의 컨텍스트 엔지니어링 가이드에서 제시하는 핵심 원칙:

> 원하는 결과의 가능성을 최대화하는, 가장 작은 고신호(high-signal) 토큰 집합을 찾아라.

이것은 백엔드 개발에서 DB 쿼리 최적화와 본질적으로 같은 사고방식이다. 필요한 데이터만, 필요한 시점에, 필요한 형태로 가져오는 것.

### 1.2 왜 프롬프트 엔지니어링으로는 부족한가

프롬프트 엔지니어링과 컨텍스트 엔지니어링의 차이는 **단일 API 호출과 시스템 아키텍처의 차이**와 같다.

| 구분 | 프롬프트 엔지니어링 | 컨텍스트 엔지니어링 |
|------|---------------------|---------------------|
| 범위 | 단일 프롬프트 텍스트 | 시스템 프롬프트 + 도구 + 메모리 + 검색 + 메타데이터 전체 |
| 시점 | 정적 (배포 시 고정) | 동적 (런타임에 조합) |
| 비유 | SQL 쿼리 하나 튜닝 | DB 스키마 + 인덱스 + 캐시 + 쿼리 전략 설계 |
| 핵심 관심사 | "어떻게 물어볼까" | "어떤 정보를 언제, 얼마나, 어떤 형태로 줄까" |

Gartner는 2026년 말까지 기업 애플리케이션의 40%가 작업 특화 AI 에이전트를 탑재할 것으로 예측했는데, 이 모든 에이전트에 컨텍스트 엔지니어링이 필요하다.

### 1.3 핵심 문제: Context Rot (컨텍스트 부패)

2025년 Chroma의 연구에서 18개 LLM 모두 컨텍스트가 길어질수록 성능이 하락했다. 일부 모델은 정확도가 95%에서 60%까지 떨어졌다. 이것이 "Context Rot"이다.

LLM의 Transformer 아키텍처는 모든 토큰 쌍의 관계를 계산(n² 복잡도)하므로, 컨텍스트가 커질수록 각 토큰에 할당되는 "주의(attention) 예산"이 줄어든다. 이는 절벽이 아니라 점진적 성능 저하(gradient)로 나타난다.

```
성능
 ^
 |■■■■■■■■■■
 |          ■■■■■
 |               ■■■■
 |                   ■■■
 |                      ■■■
 |                         ■■■■
 +-----------------------------> 컨텍스트 길이
```

백엔드 개발자에게 친숙한 비유로, 이것은 DB 테이블에 인덱스 없이 데이터를 계속 넣는 것과 같다. 데이터가 많다고 좋은 게 아니라, **잘 정리된 데이터가 좋은 것**이다.

### 1.4 컨텍스트의 4대 구성요소

#### (1) System Prompt (시스템 프롬프트)

에이전트의 "역할과 규칙"을 정의하는 부분이다. Anthropic이 권장하는 사항:

- **적절한 추상화 수준의 언어 사용**: 지나치게 세부적인 if-else 로직은 취약하고, 지나치게 추상적이면 모호하다
- **섹션 구조화**: `<background_information>`, `<instructions>`, `<tool_guidance>`, `<output_format>` 등으로 분리
- **최소 출발, 반복 개선**: 최소한의 프롬프트로 시작하고, 관찰된 실패 사례에 대응하여 추가

NestJS 개발자라면 이렇게 비유할 수 있다:

```
System Prompt = Module의 @Injectable() 데코레이터 + 설정
  - 역할 정의 = Provider의 역할
  - 행동 규칙 = Guard/Interceptor 설정
  - 출력 형식 = DTO/Pipe 변환 규칙
```

#### (2) Tools (도구/함수)

에이전트가 외부 세계와 상호작용하는 인터페이스다. 도구 설계 원칙:

- **자기 완결적(self-contained)**: 각 도구는 독립적으로 이해 가능해야 함
- **명확한 범위(clearly scoped)**: 기능 중복 최소화
- **토큰 효율적 반환**: 도구 결과가 컨텍스트를 낭비하지 않도록 설계
- **인간이 선택 못하면 AI도 못한다**: 엔지니어가 직관적으로 "이 상황엔 이 도구"를 고를 수 없다면, 도구 설계를 다시 해야 함

```typescript
// 나쁜 도구 설계: 기능 중복, 모호한 경계
tools: [
  { name: 'searchDatabase', description: '데이터베이스 검색' },
  { name: 'queryData', description: '데이터 조회' },      // searchDatabase와 뭐가 다른가?
  { name: 'findRecords', description: '레코드 찾기' },     // 또 중복
]

// 좋은 도구 설계: 명확한 역할 분리
tools: [
  { name: 'getUserById', description: '사용자 ID로 단건 조회' },
  { name: 'searchUsers', description: '조건으로 사용자 목록 검색 (name, email, role 필터)' },
  { name: 'getUserActivity', description: '특정 사용자의 최근 활동 로그 조회' },
]
```

#### (3) Retrieved Knowledge (검색된 지식 / RAG)

에이전트가 보유하지 않은 정보를 런타임에 가져오는 패턴이다. RAG(Retrieval-Augmented Generation)가 대표적이다.

**Just-in-Time 검색 전략** (Anthropic 권장):

- 모든 데이터를 미리 로드하지 말고, 가벼운 식별자(ID, 경로)만 유지
- 필요한 시점에 도구를 통해 동적으로 로드
- 인간의 인지 패턴(필요할 때 찾아보기)과 유사

**하이브리드 전략:**

- 핵심 데이터(프로젝트 규칙, 코딩 컨벤션)는 사전 로드
- 세부 데이터(특정 파일, 특정 이슈)는 탐색을 통해 발견

```typescript
// NestJS에서의 컨텍스트 빌더 패턴 예시
class ContextBuilder {
  private context: ContextBlock[] = [];
  private tokenBudget: number = 100000;
  private usedTokens: number = 0;

  // 항상 포함되는 핵심 컨텍스트 (사전 로드)
  addCoreContext(systemPrompt: string, projectRules: string) {
    this.context.push({ type: 'system', content: systemPrompt, priority: 1 });
    this.context.push({ type: 'rules', content: projectRules, priority: 2 });
  }

  // 필요 시점에 검색하여 추가 (Just-in-Time)
  async addRetrievedContext(query: string) {
    const relevant = await this.vectorStore.search(query, { topK: 5 });
    for (const doc of relevant) {
      if (this.usedTokens + doc.tokenCount < this.tokenBudget) {
        this.context.push({ type: 'retrieved', content: doc.text, priority: 3 });
        this.usedTokens += doc.tokenCount;
      }
    }
  }

  // 토큰 예산에 맞게 최적화
  build(): string {
    return this.context
      .sort((a, b) => a.priority - b.priority)
      .map(c => c.content)
      .join('\n---\n');
  }
}
```

#### (4) Conversation History & Metadata (대화 이력과 메타데이터)

파일 경로, 타임스탬프, 사용자 이름 같은 메타데이터가 에이전트의 행동에 암묵적 신호를 제공한다. 이것은 흔히 간과되지만, 에이전트가 상황을 파악하는 데 결정적 역할을 한다.

### 1.5 장기 작업을 위한 3가지 전략

에이전트가 긴 작업을 수행할 때 컨텍스트 윈도우를 관리하는 핵심 전략이다.

#### (1) Compaction (압축)

대화를 요약하고, 컨텍스트를 리셋한 뒤, 요약본으로 계속 진행하는 전략이다.

```
[대화 시작] → [작업 진행...] → [컨텍스트 80% 도달]
    → [LLM이 지금까지의 대화를 요약]
    → [컨텍스트 리셋 + 요약본만 유지]
    → [요약본 기반으로 작업 계속]
```

핵심 원칙: **재현율(Recall) 우선, 그 다음 정밀도(Precision)**. 먼저 중요한 정보를 모두 포착하는 것을 확인하고, 그 다음 불필요한 정보를 제거하는 방향으로 반복한다.

#### (2) Structured Note-Taking (구조화된 메모)

에이전트가 컨텍스트 윈도우 바깥의 영구 저장소에 메모를 작성하는 전략이다.

```typescript
// 에이전트의 scratchpad 파일 예시
interface AgentScratchpad {
  currentGoal: string;
  completedSteps: string[];
  pendingTasks: string[];
  keyDecisions: { decision: string; reason: string }[];
  discoveredFacts: { fact: string; source: string }[];
}
```

이것은 개발자가 복잡한 디버깅을 할 때 메모를 적어가며 작업하는 것과 같다.

#### (3) Sub-Agent Architecture (하위 에이전트)

복잡한 작업을 전문화된 하위 에이전트에게 위임하고, 결과만 받아오는 구조다.

```
[Coordinator Agent]
    ├── [Code Analysis Sub-Agent] → "보안 취약점 3개 발견"
    ├── [Test Generation Sub-Agent] → "테스트 케이스 12개 생성"
    └── [Documentation Sub-Agent] → "API 문서 업데이트 완료"
```

각 하위 에이전트는 자신의 컨텍스트 윈도우를 독립적으로 사용하므로, 전체 시스템의 컨텍스트 용량이 사실상 무제한으로 확장된다.

### 1.6 컨텍스트 엔지니어링 안티패턴

| 안티패턴 | 설명 | 해결책 |
|----------|------|--------|
| 과잉 프롬프트 | 하드코딩된 복잡한 if-else 로직 | 원칙 기반의 간결한 지침 |
| 도구 비대화 | 기능이 겹치는 도구 난립 | 명확한 역할 분리, 중복 제거 |
| 무차별 RAG | 관련 없는 문서까지 전부 주입 | 랭킹 + 필터링 + 토큰 예산 |
| 컨텍스트 방치 | 오래된 도구 결과를 정리하지 않음 | 주기적 정리, 결과 만료 정책 |
| 조기 최적화 | 처음부터 복잡한 컨텍스트 파이프라인 설계 | 최소 출발, 실패 관찰 후 개선 |

---

## 2. Harness Engineering (하네스 엔지니어링)

### 2.1 정의

Harness Engineering은 AI 에이전트를 프로덕션 환경에서 안정적으로 운영하기 위한 주변 인프라 전체를 설계하는 분야다. "하네스"는 말의 고삐와 안장에서 온 비유로, 강력하지만 예측 불가능한 존재를 올바른 방향으로 이끄는 장비를 뜻한다[^1].

Martin Fowler의 하네스 엔지니어링 글에서 정리된 핵심 통찰:

> 모델은 범용재(commodity)다. 하네스가 해자(moat)다.

2025년이 "에이전트가 코드를 쓸 수 있다"는 것을 증명한 해였다면, 2026년은 "에이전트가 아니라 하네스가 어려운 부분"이라는 것을 깨달은 해다. Manus는 6개월간 5번 재작성했고, LangChain은 1년간 4번의 아키텍처 변경을 거쳤다. 하네스는 한 번에 완성되지 않는다.

### 2.2 하네스의 6대 구성요소

```
┌─────────────────────────────────────────────────────┐
│                  Agent Harness                       │
│                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │   Context     │  │ Verification │  │   State    │ │
│  │ Engineering   │  │    Loops     │  │ Management │ │
│  └──────┬───────┘  └──────┬───────┘  └─────┬──────┘ │
│         │                 │                 │        │
│         ▼                 ▼                 ▼        │
│  ┌──────────────────────────────────────────────────┐│
│  │              LLM (AI Agent Core)                 ││
│  └──────────────────────────────────────────────────┘│
│         ▲                 ▲                 ▲        │
│  ┌──────┴───────┐  ┌──────┴───────┐  ┌─────┴──────┐ │
│  │    Tool      │  │  Human-in-   │  │ Lifecycle  │ │
│  │ Orchestration│  │  the-Loop    │  │ Management │ │
│  └──────────────┘  └──────────────┘  └────────────┘ │
└─────────────────────────────────────────────────────┘
```

#### (1) Context Engineering

앞서 다룬 컨텍스트 엔지니어링이 하네스의 첫 번째 구성요소다. 에이전트가 받는 입력의 품질을 결정한다.

#### (2) Verification Loops (검증 루프)

에이전트의 출력을 자동으로 검증하는 피드백 루프다. 이것이 데모와 프로덕션의 가장 큰 차이점이다.

```typescript
// 검증 루프의 계층 구조
interface VerificationPipeline {
  layers: [
    // Layer 1: 정적 검증 (빠르고 저렴)
    { type: 'syntax', check: 'lint + type-check' },

    // Layer 2: 단위 검증 (중간)
    { type: 'unit-test', check: '관련 테스트 실행' },

    // Layer 3: 통합 검증 (느리지만 포괄적)
    { type: 'integration', check: 'E2E 테스트 + 빌드' },

    // Layer 4: 의미 검증 (LLM 기반)
    { type: 'semantic', check: '다른 LLM이 결과 리뷰' },

    // Layer 5: 인간 검증 (최종)
    { type: 'human', check: '사람이 최종 승인' },
  ];
}
```

핵심 패턴은 "빠르고 저렴한 검증을 먼저, 느리고 비싼 검증을 나중에" 하는 것이다. NestJS의 Pipe → Guard → Interceptor 체인과 같은 사고방식이다.

#### (3) State Management (상태 관리)

에이전트의 실행 상태를 추적하고 관리하는 시스템이다.

```typescript
// 에이전트 상태 모델
enum AgentState {
  IDLE = 'idle',
  PLANNING = 'planning',
  EXECUTING = 'executing',
  VERIFYING = 'verifying',
  WAITING_FOR_HUMAN = 'waiting_for_human',
  RECOVERING = 'recovering',
  COMPLETED = 'completed',
  FAILED = 'failed',
}

interface AgentExecution {
  id: string;
  state: AgentState;
  task: TaskDefinition;
  steps: ExecutionStep[];      // 실행 이력
  checkpoints: Checkpoint[];   // 복구 지점
  artifacts: Artifact[];       // 생성된 결과물
  startedAt: Date;
  updatedAt: Date;
}
```

에이전트는 언제든 실패할 수 있으므로, 체크포인트 기반의 복구가 필수적이다. Spring의 트랜잭션 관리나 Saga 패턴과 같은 사고방식이다.

#### (4) Tool Orchestration (도구 오케스트레이션)

에이전트가 사용하는 도구의 등록, 권한, 에러 처리, 속도 제한을 관리한다.

```typescript
// NestJS 스타일의 도구 레지스트리
@Injectable()
class ToolRegistry {
  private tools: Map<string, ToolDefinition> = new Map();

  register(tool: ToolDefinition) {
    this.validateNoOverlap(tool);
    this.tools.set(tool.name, tool);
  }

  async execute(toolName: string, params: any, context: ExecutionContext) {
    const tool = this.tools.get(toolName);

    // 1. 권한 검사 (Guard)
    await this.checkPermission(tool, context);

    // 2. 입력 검증 (Pipe)
    const validatedParams = await this.validateParams(tool, params);

    // 3. 속도 제한 (Throttle)
    await this.rateLimiter.check(toolName);

    // 4. 실행 + 타임아웃
    const result = await Promise.race([
      tool.handler(validatedParams),
      this.timeout(tool.timeoutMs),
    ]);

    // 5. 출력 최적화 (토큰 절약)
    return this.optimizeOutput(result, context.remainingTokenBudget);
  }
}
```

#### (5) Human-in-the-Loop (사람 개입 설계)

에이전트가 자율적으로 동작하되, 특정 상황에서 사람의 판단을 요청하는 시스템이다.

설계 시 결정해야 할 3가지 질문:

1. **언제** 사람에게 물어볼 것인가? (신뢰도 임계값, 위험도 기준)
2. **어떤 형태**로 물어볼 것인가? (승인/거부, 선택지, 자유 입력)
3. 사람이 **응답하지 않으면** 어떻게 할 것인가? (대기, 안전한 기본값, 작업 중단)

```typescript
// 위험도 기반 개입 전략
class InterventionPolicy {
  shouldRequestHuman(action: AgentAction): boolean {
    if (action.riskLevel === 'high') return true;
    if (action.confidence < 0.7) return true;
    if (action.estimatedCost > this.costThreshold) return true;
    return false;
  }
}
```

#### (6) Lifecycle Management (생명주기 관리)

에이전트의 생성부터 종료까지의 전체 생명주기를 관리한다.

```
생성(Create) → 초기화(Init) → 실행(Execute) → 일시정지(Pause)
    ↓                                              ↓
  설정 로드                                      체크포인트 저장
  도구 등록                                      상태 저장
  컨텍스트 구성
    ↓                                              ↓
  실행(Resume) ← ← ← ← ← ← ← ← ← ← ← ← ← ← 재개
    ↓
  완료(Complete) 또는 실패(Fail)
    ↓
  정리(Cleanup): 임시 파일 삭제, 리소스 해제, 로그 아카이빙
```

### 2.3 하네스 아키텍처 패턴

**패턴 1: Single-Agent + Rich Harness**

가장 단순한 패턴. 하나의 에이전트에 강력한 하네스를 감싸는 구조다.

```
[Request] → [Context Builder] → [Agent] → [Verification] → [Response]
                                    ↑            ↓
                                [Tool Registry]  [State Store]
```

적합한 경우: 단일 도메인, 명확한 작업, 팀 규모가 작을 때

**패턴 2: Coordinator + Sub-Agents**

코디네이터가 작업을 분배하고, 전문화된 하위 에이전트가 실행하는 구조다.

```
[Request] → [Coordinator Agent]
                ├── [Planner Sub-Agent]
                ├── [Executor Sub-Agent]
                ├── [Reviewer Sub-Agent]
                └── [Reporter Sub-Agent]
            [Shared State Store]
```

적합한 경우: 복잡한 다단계 작업, 다양한 도구 사용, 높은 정확도 요구

**패턴 3: Pipeline Architecture**

에이전트를 파이프라인의 단계로 연결하는 구조다.

```
[Input] → [Agent 1: 분석] → [Agent 2: 계획] → [Agent 3: 실행] → [Agent 4: 검증] → [Output]
```

적합한 경우: 단계가 명확하고 순차적인 워크플로우 (CI/CD와 유사)

### 2.4 하네스 엔지니어링 5대 원칙

1. **점진적 자율성 부여 (Progressive Autonomy)**: 처음에는 모든 것을 사람이 승인하게 하고, 신뢰가 쌓이면 자율성을 점진적으로 확대한다.

2. **실패는 당연하다 (Failure is Expected)**: 에이전트는 반드시 실패한다. 하네스의 가치는 실패를 방지하는 게 아니라, 실패를 빠르게 감지하고 우아하게 복구하는 데 있다.

3. **관찰 가능성 (Observability)**: 에이전트의 모든 결정과 행동을 로깅하고 추적할 수 있어야 한다. "왜 그런 결정을 했는지" 사후에 분석 가능해야 한다.

4. **도구 설계가 하네스의 절반 (Tool Design is Half the Harness)**: 에이전트의 능력은 가용한 도구에 의해 결정된다. 잘 설계된 도구 세트는 프롬프트 최적화보다 더 큰 성능 향상을 가져온다.

5. **최소 시작, 반복 개선 (Start Simple, Iterate)**: 하네스는 한 번에 완성되지 않는다. 작게 시작하고, 실패에서 배우고, 반복적으로 개선해야 한다.

---

## 3. 두 분야의 관계

```
┌─────────────────────────────────────────┐
│         Harness Engineering             │
│                                         │
│  ┌─────────────────────────────┐        │
│  │   Context Engineering       │        │
│  │                             │        │
│  │  - System Prompt 설계       │        │
│  │  - 도구 컨텍스트 관리       │        │
│  │  - RAG / 메모리 시스템      │        │
│  │  - 토큰 최적화              │        │
│  └─────────────────────────────┘        │
│                                         │
│  + Verification Loops                   │
│  + State Management                     │
│  + Tool Orchestration                   │
│  + Human-in-the-Loop                    │
│  + Lifecycle Management                 │
│                                         │
└─────────────────────────────────────────┘
```

Context Engineering은 Harness Engineering의 핵심 하위 구성요소다. 컨텍스트 엔지니어링이 "에이전트가 올바른 정보를 받게 하는 것"에 집중한다면, 하네스 엔지니어링은 "에이전트가 올바른 정보를 받고, 올바르게 행동하고, 실패 시 복구되고, 안전하게 운영되는 전체 시스템"을 설계한다.

---

## 4. 백엔드 개발자의 강점

NestJS/Spring 개발자는 이 분야에서 자연스러운 강점을 갖고 있다.

| 백엔드 개발 경험 | 하네스 엔지니어링 적용 |
|-----------------|----------------------|
| DI (의존성 주입) | 도구 레지스트리, 컨텍스트 빌더 조립 |
| Guard / Interceptor | 검증 루프, 가드레일 미들웨어 |
| Pipe / DTO 변환 | 도구 입출력 정규화, 토큰 최적화 |
| Queue (Bull/Redis) | 에이전트 작업 큐잉, 비동기 실행 |
| 트랜잭션 / Saga | 에이전트 상태 관리, 체크포인트 복구 |
| 마이크로서비스 | Sub-Agent 아키텍처, 서비스 간 통신 |
| API 버저닝 | 도구 버저닝, 하위 호환성 관리 |

---

## 마치며

프롬프트 엔지니어링이 "LLM에게 좋은 질문을 하는 기술"이었다면, 컨텍스트 엔지니어링은 "LLM이 좋은 판단을 내릴 수 있는 환경을 만드는 기술"이고, 하네스 엔지니어링은 "그 판단이 프로덕션에서 안정적으로 동작하도록 전체 시스템을 설계하는 기술"이다.

이 세 가지는 배척 관계가 아니라 포함 관계이며, AI 에이전트 시대의 소프트웨어 엔지니어에게 필요한 스킬이 단계적으로 확장되고 있다는 것을 보여준다.

특히 백엔드 개발자라면 이미 익숙한 개념들 — DI, 미들웨어 체인, 트랜잭션, 큐, 마이크로서비스 — 이 하네스 엔지니어링의 핵심 구성요소와 정확히 대응된다는 점에서, 이 분야로의 전환 비용이 생각보다 낮다.

<!-- 내 의견: -->

---

## 참고 자료

[^1]: OpenAI, "Harness Engineering: Leveraging Codex in an Agent-First World" (2026)

- Anthropic, "Effective Context Engineering for AI Agents" (2025)
- Martin Fowler, "Harness Engineering for Coding Agent Users" (2026)
- ByteByteGo, "A Guide to Context Engineering for LLMs"
- State of Context Engineering in 2026 - SwirlAI Newsletter
- LogRocket, "The LLM Context Problem in 2026"
- Aakash Gupta, "2025 Was Agents. 2026 Is Agent Harnesses."
- CodeConductor, "Context Engineering: A Complete Guide (2026)"
- GitHub, "awesome-harness-engineering" (커뮤니티 리소스 모음)
- Arxiv, "Agentic Context Engineering: Evolving Contexts for Self-Improving Language Models"
