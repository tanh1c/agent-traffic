# TrafficFlow Agent — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Hoàn thành proposal document (stage 1) + demo prototype cho GDGoC Hackathon Vietnam 2026

**Architecture:** Proposal gồm 2 phần chính: (1) Written document theo template + (2) Demo interactive prototype thể hiện hệ thống hoạt động

**Tech Stack:** Next.js (dashboard demo) + YOLO v8 + Multi-provider LLM API (Groq/OpenRouter/Alibaba) + Simulation Engine + Google Maps Traffic API

---

## Task 1: Hoàn thành Proposal Document

**Files:**
- Modify: `Ideas/proposal.md`

**Step 1: Điền thông tin team**

Điền vào phần Team Information:
- Team Name
- Tên 3-4 thành viên + vai trò

**Step 2: Viết Project Overview**

- Project Name: TrafficFlow Agent
- Elevator Pitch: problem → solution → audience → value
- Why Agentic AI: Goal + Planning + Autonomy + Feedback Loop

**Step 3: Viết Problem & Solution**

- Problem Statement: TP.HCM, khu vực Trường Chinh – Cách Mạng Tháng 8, đèn cố định không thích ứng
- Proposed Solution: Hybrid congestion detection (density + speed + queue length) + LLM reasoning
- Unique Selling Point: Kết hợp camera real-time + Google Maps — vượt trội Google Maps thuần GPS

**Step 4: Viết Technology Stack**

- LLM: Groq / OpenRouter / Alibaba Qwen (multi-provider)
- Agent Framework: LangChain / LangGraph
- Object Detection: YOLO v8
- Map Data: Google Maps Traffic API + OSRM
- Simulation: Custom web engine
- Frontend: Next.js / React
- Cloud: Google Cloud Platform

**Step 5: Viết Feasibility & Development Plan**

- Development Plan: Day 1 → Day 2 → Demo
- Budget estimate

**Step 6: Viết Extensions (Future Work)**

- B) Phương tiện ưu tiên
- C) Dự báo ùn tắt trước
- D) Dashboard CSGT

---

## Task 2: Setup Next.js Project cho Simulation Demo

**Files:**
- Create: `traffic-sim/` (Next.js project)

**Step 1: Setup Next.js project**

```bash
cd Ideas
npx create-next-app@latest traffic-sim --typescript --tailwind --app
cd traffic-sim && npm install
```

**Step 2: Tạo folder structure**

```
traffic-sim/
  src/
    components/
      MapView.tsx
      TrafficLight.tsx
      Vehicle.tsx
      Dashboard.tsx
      MetricsPanel.tsx
    services/
      mapsService.ts
      llmService.ts
      simulationEngine.ts
    agent/
      congestionDetector.ts
      llmProvider.ts
      decisionEngine.ts
    types/
      index.ts
```

**Step 3: Tạo TypeScript types**

```typescript
export interface Vehicle {
  id: string;
  type: 'car' | 'truck' | 'bike' | 'bus';
  lane: number;
  position: { x: number; y: number };
  speed: number;
}

export interface LaneMetrics {
  laneId: string;
  vehicleCount: number;
  density: number;       // xe/don vi dien tich
  avgSpeed: number;      // km/h
  queueLength: number;   // mét
  waitTime: number;      // giây
}

export interface AgentDecision {
  action: 'extend_green' | 'reduce_green' | 'all_red' | 'no_change';
  targetArm: 'N' | 'S' | 'E' | 'W';
  duration: number;      // giây
  reasoning: string;
  severity: 'normal' | 'warning' | 'critical';
}

export interface IntersectionState {
  arms: Record<'N' | 'S' | 'E' | 'W', {
    greenDuration: number;
    redDuration: number;
    currentPhase: 'green' | 'yellow' | 'red';
    metrics: LaneMetrics;
  }>;
}
```

---

## Task 3: Xây dựng Simulation Engine + Map Visualization

**Files:**
- Create: `traffic-sim/src/components/MapView.tsx`
- Create: `traffic-sim/src/services/simulationEngine.ts`

**Step 1: Tạo MapView component**

- SVG canvas vẽ bản đồ ngã tư Trường Chinh – Cách Mạng Tháng 8
- 4 nhánh: North (Trường Chinh northbound), South, East (Cách Mạng Tháng 8), West
- Mỗi nhánh 2-3 làn xe
- Vẽ intersection point ở giữa

**Step 2: Tạo Vehicle component + movement logic**

```typescript
// Simulation tick: mỗi 100ms
function simulateTick() {
  vehicles.forEach(vehicle => {
    if (canMove(vehicle)) {
      vehicle.position = moveForward(vehicle);
    } else {
      vehicle.waitTime += 100;
    }
  });
}
```

**Step 3: Tạo TrafficLight component**

- Hiển thị đèn xanh/vàng/đỏ cho mỗi nhánh
- Nhận input: currentPhase, duration
- Animation chuyển màu mượt

**Step 4: Chạy baseline simulation (Agent OFF)**

- Cycle cố định: mỗi nhánh 30s green, 5s yellow, chuyển vòng
- Record metrics: wait time, queue length, throughput
- Đây là baseline để so sánh với Agent ON

---

## Task 4: Mock YOLO Detection Layer

**Files:**
- Create: `traffic-sim/src/services/mockYolo.ts`
- Modify: `traffic-sim/src/services/simulationEngine.ts`

**Step 1: Tạo mock vehicle generator**

```typescript
function generateMockVehicles(intensity: 'low' | 'medium' | 'high'): Vehicle[] {
  const counts = { low: 5, medium: 15, high: 30 };
  return Array.from({ length: counts[intensity] }, (_, i) => ({
    id: `v${i}`,
    type: randomType(),
    lane: randomLane(),
    position: randomPosition(),
    speed: randomSpeed()
  }));
}
```

**Step 2: Tạo congestion detector**

```typescript
function detectCongestion(laneMetrics: LaneMetrics): 'clear' | 'moderate' | 'heavy' | 'gridlock' {
  const score = laneMetrics.density * 0.4 + (50 - laneMetrics.avgSpeed) * 0.3 + laneMetrics.queueLength / 10 * 0.3;
  if (score < 20) return 'clear';
  if (score < 40) return 'moderate';
  if (score < 60) return 'heavy';
  return 'gridlock';
}
```

**Step 3: Tích hợp vào simulation tick**

- Mỗi tick: detect congestion → update metrics → feed to agent

---

## Task 5: Xây dựng Multi-Provider LLM Agent

**Files:**
- Create: `traffic-sim/src/agent/llmProvider.ts`
- Create: `traffic-sim/src/agent/decisionEngine.ts`

**Step 1: Tạo LLM provider abstraction**

```typescript
type LLMProvider = 'groq' | 'openrouter' | 'alibaba';

interface LLMConfig {
  provider: LLMProvider;
  apiKey: string;
  model: string;
  baseUrl: string;
}

async function callLLM(prompt: string, config: LLMConfig): Promise<string> {
  const response = await fetch(config.baseUrl + '/chat/completions', {
    method: 'POST',
    headers: { 'Authorization': `Bearer ${config.apiKey}`, 'Content-Type': 'application/json' },
    body: JSON.stringify({ model: config.model, messages: [{ role: 'user', content: prompt }] })
  });
  const data = await response.json();
  return data.choices[0].message.content;
}
```

**Step 2: Tạo Agent prompt template**

```typescript
const AGENT_PROMPT = `
Bạn là TrafficFlow Agent — chuyên gia điều phối giao thông.
Trạng thái hiện tại tại ngã tư:
- Hướng Bắc: {N_density} xe, tốc độ {N_speed} km/h, queue {N_queue}m, congestion: {N_status}
- Hướng Nam: {S_density} xe, tốc độ {S_speed} km/h, queue {S_queue}m, congestion: {S_status}
- Hướng Đông: {E_density} xe, tốc độ {E_speed} km/h, queue {E_queue}m, congestion: {E_status}
- Hướng Tây: {W_density} xe, tốc độ {W_speed} km/h, queue {W_queue}m, congestion: {W_status}
Thời gian: {time} | Ngày trong tuần: {day}
Ngưỡng: density>25, speed<20km/h, queue>100m = ùn tắt

Mục tiêu: Giảm thời gian chờ trung bình, tránh gridlock.
Đề xuất điều chỉnh đèn tín hiệu (áp dụng ngay cho cycle tiếp theo).
Trả lời JSON: { "action": "extend_green|reduce_green|all_red|no_change", "target": "N|S|E|W", "duration": 15-45, "reasoning": "...", "severity": "normal|warning|critical" }
`;
```

**Step 3: Implement decision engine với fallback**

```typescript
async function getAgentDecision(metrics: Record<string, LaneMetrics>): Promise<AgentDecision> {
  const prompt = fillPrompt(AGENT_PROMPT, metrics);

  const providers = [
    { provider: 'groq', config: GROQ_CONFIG },
    { provider: 'openrouter', config: OPENROUTER_CONFIG },
    { provider: 'alibaba', config: ALIBABA_CONFIG }
  ];

  for (const { provider, config } of providers) {
    try {
      const response = await callLLM(prompt, config);
      return parseJSONResponse(response);
    } catch (e) {
      console.warn(`Provider ${provider} failed, trying next...`);
    }
  }

  // Fallback: rule-based decision
  return ruleBasedDecision(metrics);
}
```

**Step 4: Implement human-in-the-loop cho case nghiêm trọng**

```typescript
if (decision.severity === 'critical') {
  showPendingApproval(decision, countdown=30);
  // Auto-apply after 30s if no human response
}
```

---

## Task 6: Hoàn thiện Dashboard CSGT

**Files:**
- Create: `traffic-sim/src/components/Dashboard.tsx`
- Create: `traffic-sim/src/components/MetricsPanel.tsx`
- Create: `traffic-sim/src/components/AgentStatusPanel.tsx`
- Create: `traffic-sim/src/components/PendingApproval.tsx`

**Step 1: MapView với traffic overlay**

- Bản đồ interactive + colored overlay theo mật độ (green/yellow/red)
- Vehicle icons di chuyển real-time
- Signal status indicator per arm

**Step 2: MetricsPanel**

```
┌────────────────────────────────┐
│  LIVE METRICS                  │
│  Avg wait: 45s  (↓12s)        │
│  Queue: 180m   (↓30m)          │
│  Throughput: 120 v/min (↑15%)  │
│  Agent: ON 🟢                   │
└────────────────────────────────┘
```

**Step 3: AgentStatusPanel**

- Online/Offline indicator
- Mode: AUTO | PENDING_APPROVAL | MANUAL
- Last action + timestamp

**Step 4: PendingApproval component**

- Hiện khi severity = warning/critical
- Mô tả tình huống + đề xuất agent
- Buttons: [Approve] [Reject] [Modify]
- Countdown timer 30s → auto apply

---

## Task 7: Tích hợp Google Maps Traffic API

**Files:**
- Create: `traffic-sim/src/services/mapsService.ts`

**Step 1: Setup Google Cloud project**

- Enable Maps JavaScript API + Directions API
- Lấy API key

**Step 2: Tạo mapsService**

```typescript
async function getTrafficData(lat: number, lng: number) {
  // Google Maps Traffic Layer → speed per segment
  // Trả về: { segment: string, speed: number, congestion: 'low'|'medium'|'high' }[]
}
```

**Step 3: Merge với YOLO data**

```typescript
function mergeTrafficData(mapsData: MapsSegment[], yoloData: LaneMetrics[]) {
  // Maps data cho tầm nhìn tổng thể (toàn tuyến)
  // YOLO data cho chi tiết cục bộ (từng làn)
  // Ghép lại → bức tranh toàn diện
}
```

**Step 4: Mock fallback**

- Khi không có API key: dùng mock traffic data tương đương

---

## Task 8: A/B Test Evaluation

**Files:**
- Create: `traffic-sim/src/services/evaluation.ts`

**Step 1: Record baseline (Agent OFF)**

```typescript
async function runBaselineSimulation(): Promise<Metrics> {
  agentEnabled = false;
  resetSimulation();
  runSimulation(duration=SIM_DURATION); // 5-10 phút simulated time
  return collectMetrics();
}
```

**Step 2: Run with Agent (Agent ON)**

```typescript
async function runAgentSimulation(): Promise<Metrics> {
  agentEnabled = true;
  resetSimulation();
  runSimulation(duration=SIM_DURATION);
  return collectMetrics();
}
```

**Step 3: Generate comparison report**

```typescript
function generateReport(baseline: Metrics, agentOn: Metrics) {
  return {
    avgWaitTime: { before: baseline.avgWait, after: agentOn.avgWait, improvement: '...' },
    queueLength: { before: baseline.queue, after: agentOn.queue, improvement: '...' },
    throughput: { before: baseline.throughput, after: agentOn.throughput, improvement: '...' }
  };
}
```

---

## Task 9: Chuẩn bị Slide Deck + Demo Video

**Step 1: Slide deck theo proposal template**

- Slide 1: Cover — "TrafficFlow Agent" + team name
- Slide 2: Problem — ùn tắt TP.HCM, số liệu thực tế
- Slide 3: Solution — Agentic AI overview
- Slide 4: Agent Architecture — Observe → Reason → Decide → Act
- Slide 5: Technology Stack — YOLO + LLM + Maps
- Slide 6: Demo screenshot — dashboard + metrics
- Slide 7: A/B Test Results — số liệu cụ thể
- Slide 8: Extensions — B, C, D
- Slide 9: Feasibility — Development plan, budget
- Slide 10: Team

**Step 2: Demo video (2-3 phút)**

- Dashboard live → agent phát hiện ùn tắt → LLM reasoning hiện lên → đèn điều chỉnh → metrics cải thiện
- Recording: dùng OBS hoặc built-in screen recorder

---

*Plan created: 2026-03-24*
*Brainstorming: completed | Design doc: saved | Implementation plan: saved*
