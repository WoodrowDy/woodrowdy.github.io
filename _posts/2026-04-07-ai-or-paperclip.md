---
title: "AI 에이전트를 회사처럼 굴려봤습니다 | Paperclip 실전 후기"
date: 2026-04-07 12:00:00 +0900
categories: [AI Engineering, PaperClip]
tags: []
---

# AI 에이전트를 회사처럼 굴려봤습니다 | Paperclip 실전 후기

## TL;DR

Paperclip은 AI 에이전트를 조직 구조로 관리할 수 있는 오픈소스 플랫폼입니다. [00:52] CEO, CTO, CMO 등 역할별 에이전트를 구성하고, 티켓 시스템으로 업무를 할당하며, 각 에이전트의 토큰 사용량과 작업 이력을 대시보드에서 시각화합니다. [00:22] 기존 OpenCLO의 텔레그램 방식이 세션 관리가 안 되던 문제를 조직도와 워크플로우 기반으로 해결한 접근법입니다.

## 영상이 다룬 핵심 개념

**멀티 에이전트 오케스트레이션 (Multi-Agent Orchestration)** [01:48]  
Paperclip의 핵심은 단일 에이전트가 아니라 여러 에이전트가 계층 구조(CEO → COO/CTO/CMO)를 이루며 협업하는 방식입니다. 영상 작성자는 CEO에 Claude Opus 4.6, 하위 에이전트에 GPT-5.4를 할당해 비용과 성능을 절충했습니다. [02:27]

**티켓 기반 워크플로우 (Ticket-based Workflow)** [01:22]  
업무는 티켓으로 생성되고, 스레드 단위로 과정이 기록됩니다. CEO가 업무를 받으면 적절한 하위 에이전트에게 할당하고, 해당 에이전트는 작업 후 결과를 상위에 보고합니다. [08:48] 이는 기업의 그룹웨어(결재/수신함) 구조와 유사합니다. [10:41]

**스킬 기반 확장 (Skill Marketplace)** [03:49]  
`skills.sh`와 같은 외부 저장소에서 스킬을 복사해 에이전트에 붙일 수 있습니다. 코드를 작성할 필요 없이 Paperclip 내장 스킬 + 커스텀 스킬로 에이전트의 능력을 확장합니다. [05:49]

**멀티 컴퍼니 모드 (Multi-Company Mode)** [01:20]  
하나의 Paperclip 인스턴스에서 여러 "회사"를 만들 수 있습니다. 영상에서는 "생산적 생산자"와 "The PKM(제텔카스텐 전용)" 두 조직을 따로 운영하는 예시를 보여줍니다. [04:24]

**대시보드 기반 관찰성 (Dashboard Observability)** [02:42]  
진행 중인 티켓, 각 에이전트별 토큰 사용량(Budget), 작업 히스토리를 웹 UI로 확인할 수 있습니다. [05:55] 모바일 브라우저 접속도 가능해 텔레그램 없이 외부에서 관리할 수 있습니다. [09:28]

## 이론적 배경

**Multi-Agent System (MAS)**  
단일 LLM이 모든 역할을 수행하는 대신, 특화된 에이전트 여러 개가 협업하는 구조입니다. 각 에이전트는 자신의 컨텍스트와 역할(Role Prompting)을 갖고, 상위 에이전트가 조정자(Orchestrator) 역할을 맡습니다. Paperclip의 CEO → COO/CTO/CMO 계층은 전형적인 Hierarchical Multi-Agent 패턴입니다.

**Tool Use & Skill Composition**  
에이전트가 외부 API나 파일 시스템에 접근할 수 있도록 "스킬"을 제공하는 방식입니다. LangChain의 Tool, OpenAI의 Function Calling과 동일한 개념이지만, Paperclip은 마켓플레이스 형태로 스킬을 공유합니다.

**Workflow Orchestration**  
티켓/스레드 기반 구조는 BPM(Business Process Management) 개념을 LLM 워크플로우에 적용한 것입니다. 각 티켓은 상태(Open → In Progress → Closed)를 가지며, 에이전트 간 메시지 패싱(Message Passing)으로 작업이 전달됩니다.

**Role-Based Prompting**  
CEO, CTO, CMO 같은 역할명은 단순한 라벨이 아니라, 각 에이전트의 시스템 프롬프트에 반영됩니다. "당신은 기술 결정을 내리는 CTO입니다"라는 맥락이 에이전트의 응답 스타일을 결정합니다.

## 기존 솔루션 / 라이브러리와의 비교

| 특징 | Paperclip | LangGraph | AutoGen | OpenCLO |
|------|-----------|-----------|---------|---------|
| **조직 구조** | 계층형 조직도 (CEO → 하위) | 그래프 기반 노드 | Conversational Agents | 단일 봇 |
| **세션 관리** | 티켓/스레드 단위 | State Graph | GroupChat 기록 | 텔레그램 채팅 (세션 약함) |
| **시각화** | 웹 대시보드 내장 | 외부 툴 필요 | 외부 툴 필요 | 없음 |
| **비용 추적** | Budget 탭에서 토큰 집계 | 수동 구현 필요 | 수동 구현 필요 | API 로그로만 확인 |
| **확장성** | 스킬 마켓플레이스 | Custom Tool 코딩 | Function Calling | 봇 코드 직접 수정 |
| **배포** | npx 1줄 로컬 실행 | 코드 프로젝트 | 코드 프로젝트 | 텔레그램 봇 서버 |

**Paperclip의 차별점**  
- **No-Code 조직 설계**: 코드 없이 CEO를 생성하고, CEO가 하위 에이전트를 생성하는 재귀적 구조 [06:10]  
- **승인 워크플로우**: 에이전트가 새 봇을 만들거나 작업을 시작할 때 사람이 "Approve/Confirm" 가능 [07:18]  
- **멀티 컴퍼니**: 하나의 인스턴스로 프로젝트별 독립 조직 운영 [01:20]

**약점**  
- 서버 안정성: 영상 중 "조금씩 에러가 뜰 때가 있다" 언급 [08:21]  
- 비용 추적 제한: OAuth 방식 사용 시 Spent가 집계 안 됨 [03:03]  
- 코드 커스터마이징 제약: LangGraph처럼 워크플로우 로직을 직접 짤 수 없음

## 시스템 관점의 고려사항

**비용 (Cost)**  
[02:27] CEO는 Claude Opus 4.6, 하위는 GPT-5.4/Codex로 차등 배치해 비용을 절감했습니다. 단, 모든 에이전트가 동시에 돌면 토큰 소비가 급증할 수 있습니다. 특히 CEO가 여러 하위 에이전트에게 동시 할당 시 Context Window가 각각 소모됩니다.  
**트레이드오프**: CEO를 똑똑하게 → 비용 ↑, 하위를 저렴 모델로 → 품질 ↓

**지연 (Latency)**  
[06:18] 영상에서 "시간이 좀 걸리기 때문에 이 사이에 딴 일을 하고 있습니다"라고 언급합니다. 에이전트 간 메시지 패싱은 순차적이므로 CEO → CTO → 하위 봇으로 갈수록 응답 시간이 누적됩니다.  
**트레이드오프**: 계층 깊이 ↑ → 정확도/맥락 ↑, 응답 시간 ↑

**신뢰성 (Reliability)**  
[08:21] 서버 에러가 간헐적으로 발생한다는 언급이 있습니다. 로컬 실행이므로 네트워크/API 타임아웃에 취약할 수 있습니다. 또한 에이전트가 "새 봇을 만들기로 결정"하면 사람이 승인해야 하므로 완전 자율 실행은 아닙니다.  
**트레이드오프**: 승인 단계 ↑ → 신뢰성 ↑, 자동화 ↓

**운영 (Operation)**  
- **로깅**: 티켓별 스레드가 자동 기록되므로 Audit Trail 확보 가능 [01:30]  
- **토큰 추적**: Budget 탭으로 에이전트별 사용량 확인 가능 [02:54] (단, OAuth 방식은 제외)  
- **캐싱**: 영상에서 명시 안 됨. LLM API 레벨 캐싱 외 별도 레이어 없는 것으로 보임  
- **모바일 접근**: 웹 UI가 반응형이므로 외부에서 승인/모니터링 가능 [09:28]

**확장성 (Scalability)**  
하위 에이전트를 계속 추가할 수 있지만, CEO의 Context Window에 모든 하위 봇 정보가 들어가야 하므로 조직 규모가 커지면 Context Overflow 위험이 있습니다.  
**해결 방향**: 중간 관리자(Manager) 레이어 추가, 에이전트별 독립 Context 관리

## 코드로 이해하기

Paperclip은 No-Code 툴이지만, 내부적으로는 다음과 같은 패턴을 따른다고 추론할 수 있습니다 (영상 내용 기반 재구성).

### 에이전트 간 메시지 패싱 (의사코드)

```typescript
// CEO가 티켓을 받으면 적절한 하위 에이전트에게 할당
class CEOAgent {
  async handleTicket(ticket: Ticket) {
    const analysis = await this.llm.call({
      role: "CEO",
      prompt: `티켓 내용: ${ticket.description}. 어느 부서가 처리해야 하나?`,
    });
    
    if (analysis.assignee === "CTO") {
      await this.delegateToAgent("CTO", ticket);
    } else if (analysis.assignee === "CMO") {
      await this.delegateToAgent("CMO", ticket);
    }
  }
  
  async delegateToAgent(role: string, ticket: Ticket) {
    const agent = this.findAgentByRole(role);
    await agent.execute(ticket);
    // 결과를 티켓 스레드에 기록
  }
}
```

이 패턴은 **LangGraph의 Conditional Edge**와 유사합니다. CEO가 라우터 역할을 하고, 조건부로 다음 노드(에이전트)를 선택합니다.

### 승인 워크플로우 (의사코드)

```typescript
// 에이전트가 새 봇 생성을 제안하면 사람이 승인할 때까지 대기
class AgentCreationWorkflow {
  async proposeNewAgent(proposal: AgentProposal) {
    await this.sendToInbox(proposal); // [07:03] Inbox에 승인 요청 등록
    
    const approved = await this.waitForHumanApproval(); // 블로킹
    
    if (approved) {
      const newAgent = await this.createAgent(proposal.config);
      await this.addToOrganization(newAgent);
    }
  }
}
```

이는 **Human-in-the-Loop (HITL)** 패턴입니다. LangGraph의 `interrupt` 기능과 유사하게 사람의 승인이 있어야 다음 단계로 진행됩니다.

## 내 의견

<!-- TODO: 본인이 직접 채울 것. AI는 이 섹션을 채우지 마세요. -->

---

> 본 글은 [AI 에이전트를 회사처럼 굴려봤습니다 | Paperclip 실전 후기](https://www.youtube.com/watch?v=Jr7IcBa0Ik8) (채널: 생산적생산자)을 시청 후 작성한 학습 노트입니다.  
> 영상의 모든 권리는 원저작자에게 있습니다.