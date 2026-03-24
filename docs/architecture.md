# TrafficFlow Agent — System Architecture

> Mermaid diagrams for GDGoC Hackathon Vietnam 2026 | Stage 1: Proposal

---

## 1. High-Level System Architecture

```mermaid
flowchart TB
    subgraph INPUT["📥 DATA INPUT LAYER"]
        CAMERA["📹 Camera Streams<br/>(YOLO v8)"]
        MAPS["🗺️ Google Maps Traffic API"]
        WEATHER["🌦️ Weather API"]
        CALENDAR["📅 Calendar API<br/>(Holiday/Events)"]
    end

    subgraph PROCESS["⚙️ PROCESSING LAYER"]
        YOLO["🔍 YOLO Processor<br/>Count · Classify · Track"]
        MERGE["🔗 Data Fusion<br/>Camera + Maps + Context"]
        CONGESTION["🚦 Congestion Detector<br/>Density + Speed + Queue"]
    end

    subgraph AGENT["🤖 AI AGENT LAYER"]
        OBSERVE["👁️ OBSERVE<br/>Collect metrics"]
        REASON["🧠 REASON<br/>LLM Analysis"]
        DECIDE["⚖️ DECIDE<br/>Action Selection"]
        ACT["▶️ ACT<br/>Apply Signal Change"]
        FEEDBACK["🔄 FEEDBACK<br/>Loop back"]
    end

    subgraph OUTPUT["📤 OUTPUT LAYER"]
        SIM["🖥️ Simulation Engine<br/>(Demo)"]
        SCATS["🚦 SCATS/ITMS API<br/>(Future)"]
        DASHBOARD["📊 CSGT Dashboard<br/>Real-time Monitoring"]
    end

    CAMERA --> YOLO
    MAPS --> MERGE
    WEATHER --> MERGE
    CALENDAR --> MERGE
    YOLO --> MERGE
    MERGE --> CONGESTION
    CONGESTION --> OBSERVE
    OBSERVE --> REASON
    REASON --> DECIDE
    DECIDE --> ACT
    ACT --> FEEDBACK
    FEEDBACK --> OBSERVE
    ACT --> SIM
    ACT --> SCATS
    ACT --> DASHBOARD
```

---

## 2. Agent Decision Loop (Core Agentic Cycle)

```mermaid
flowchart LR
    subgraph OBSERVE["👁️ OBSERVE"]
        O1["Camera: Count vehicles<br/>Speed per lane<br/>Queue length"]
        O2["Maps: Speed per segment<br/>Congestion color<br/>Incidents"]
        O3["Compute: Congestion score<br/>= density×0.4 + (50-speed)×0.3 + queue×0.3"]
    end

    subgraph REASON["🧠 REASON (LLM)"]
        R1["Build prompt:<br/>[Metrics] + [Goal] + [Thresholds]"]
        R2["Call LLM (Groq → OpenRouter → Alibaba)"]
        R3["Parse JSON response:<br/>action + target + duration + reasoning"]
    end

    subgraph DECIDE["⚖️ DECIDE"]
        D1{"Severity?"}
        D2["NORMAL<br/>→ Auto-apply"]
        D3["WARNING<br/>→ Dashboard alert"]
        D4["CRITICAL<br/>→ Human approval"]
    end

    subgraph ACT["▶️ ACT"]
        A1["Send signal command<br/>to simulation/SCATS"]
        A2["Update traffic light<br/>green/red duration"]
    end

    OBSERVE --> REASON
    REASON --> DECIDE
    D1 -->|"normal"| D2
    D1 -->|"warning"| D3
    D1 -->|"critical"| D4
    D2 --> ACT
    D3 --> DASHBOARD["📋 Dashboard<br/>Notify CSGT"]
    D4 --> DASHBOARD
    DASHBOARD -->|"30s timeout<br/>or approve"| ACT
    ACT --> FEEDBACK["🔄 FEEDBACK<br/>Re-observe traffic state"]
    FEEDBACK --> OBSERVE
```

---

## 3. Data Flow Architecture

```mermaid
flowchart TB
    subgraph CAMERA_LAYER["📹 CAMERA LAYER"]
        C1["Public Traffic Cameras<br/>(RTSP/HTTP streams)"]
        C2["Synthetic Data<br/>(Mock · UA-DETRAC · BDD100K)"]
    end

    subgraph PROCESSING["⚡ PROCESSING (Edge/Cloud)"]
        YOLO["YOLO v8<br/>Object Detection"]
        TRACK["Object Tracking<br/>Vehicle Speed Estimation"]
        COUNT["Vehicle Counter<br/>Per-lane density"]
    end

    subgraph MAPS_LAYER["🗺️ MAPS LAYER"]
        GMAPS["Google Maps<br/>Traffic API"]
        OSRM["OSRM<br/>(Backup)"]
    end

    subgraph FUSION["🔗 DATA FUSION"]
        F1["Merge Camera data<br/>(local detail)"]
        F2["Merge Maps data<br/>(global view)"]
        F3["Add Context:<br/>Time · Weather · Events"]
    end

    subgraph OUTPUT["📊 OUTPUTS"]
        METRICS["Real-time Metrics<br/>Dashboard"]
        DECISION["Agent Decision<br/>→ Signal Control"]
        HISTORY["Historical Log<br/>→ A/B Evaluation"]
    end

    C1 --> YOLO
    C2 --> YOLO
    YOLO --> TRACK
    TRACK --> COUNT
    COUNT --> F1
    GMAPS --> F2
    OSRM --> F2
    F1 --> F3
    F2 --> F3
    F3 --> METRICS
    F3 --> DECISION
    DECISION --> HISTORY
```

---

## 4. Congestion Detection Algorithm

```mermaid
flowchart TD
    START["Start Detection Loop<br/>Every 1 second"]
    INPUT["Get lane metrics:<br/>• vehicleCount<br/>• avgSpeed<br/>• queueLength"]
    CONTEXT["Get context:<br/>• timeOfDay<br/>• dayOfWeek<br/>• weather"]
    CALC["Calculate congestion score"]
    SCORE{"Score"}
    CLEAR["Status: CLEAR<br/>Green time: default"]
    MODERATE["Status: MODERATE<br/>Green time: +10s"]
    HEAVY["Status: HEAVY<br/>Green time: +20s<br/>Alert: WARNING"]
    GRIDLOCK["Status: GRIDLOCK<br/>Green time: +30s<br/>Alert: CRITICAL<br/>Request human approval"]

    START --> INPUT
    INPUT --> CONTEXT
    CONTEXT --> CALC

    CALC["Score =<br/>density × 0.4<br/>+ (50 - speed) × 0.3<br/>+ queue/10 × 0.3"]

    SCORE -->|"< 20"| CLEAR
    SCORE -->|"20–40"| MODERATE
    SCORE -->|"40–60"| HEAVY
    SCORE -->|"> 60"| GRIDLOCK

    style CLEAR fill:#bbf7d0,stroke:#16a34a
    style MODERATE fill:#fef08a,stroke:#ca8a04
    style HEAVY fill:#fed7aa,stroke:#ea580c
    style GRIDLOCK fill:#fecaca,stroke:#dc2626
```

---

## 5. LLM Multi-Provider Architecture

```mermaid
flowchart TB
    PROMPT["📝 Build Agent Prompt<br/>[Metrics] + [Goal] + [JSON schema]"]

    PROVIDERS{"Try Providers in Order"}

    GROQ["🥇 Groq API<br/>llama-3.3-70b<br/>Fast · Cheap"]
    OPENROUTER["🥈 OpenRouter<br/>gpt-4o-mini / qwen<br/>Fallback"]
    ALIBABA["🥉 Alibaba Qwen<br/>qwen-turbo<br/>Last resort"]

    PARSE["📤 Parse JSON Response<br/>{action, target, duration, severity}"]
    VALID{"Valid JSON?"}
    RULE["⚡ Rule-Based Fallback<br/>Simple threshold logic"]
    OUTPUT["✅ Agent Decision Ready"]

    PROMPT --> PROVIDERS
    PROVIDERS -->|"1st"| GROQ
    PROVIDERS -->|"2nd"| OPENROUTER
    PROVIDERS -->|"3rd"| ALIBABA
    GROQ -->|"success"| PARSE
    OPENROUTER -->|"groq failed"| PARSE
    ALIBABA -->|"all failed"| PARSE
    PARSE --> VALID
    VALID -->|"no"| RULE
    VALID -->|"yes"| OUTPUT
    RULE --> OUTPUT

    style GROQ fill:#bbf7d0,stroke:#16a34a
    style OUTPUT fill:#dbeafe,stroke:#2563eb
    style RULE fill:#fef3c7,stroke:#d97706
```

---

## 6. Dashboard Layout

```mermaid
flowchart TB
    subgraph HEADER["🚦 TrafficFlow Agent Dashboard"]
        H1["🟢 Agent Status: ONLINE"]
        H2["Mode: AUTO"]
        H3["Last Action: +20s green N"]
    end

    subgraph BODY["📊 Main Content"]
        subgraph LEFT["🗺️ Left Panel: Map View"]
            M1["Interactive Map<br/>Trường Chinh – Cách Mạng Tháng 8"]
            M2["Traffic overlay<br/>🟢 Yellow 🔴"]
            M3["🚦 Signal indicators<br/>per arm (N/S/E/W)"]
            M4["🚗 Vehicle animation<br/>real-time movement"]
        end

        subgraph MIDDLE["📈 Middle Panel: Live Metrics"]
            MET1["⏱️ Avg Wait Time"]
            MET2["📏 Queue Length"]
            MET3["🚗 Throughput"]
            MET4["🔴 Incidents: 0"]
        end

        subgraph RIGHT["🤖 Right Panel: Agent Status"]
            A1["● Online"]
            A2["Mode: AUTO<br/>PENDING_APPROVAL<br/>MANUAL"]
            A3["LLM Provider: Groq"]
            A4["Last Decision: reason here..."]
        end
    end

    subgraph FOOTER["📋 Bottom: Pending Approvals"]
        P1["⚠️ Critical Alert"]
        P2["Accident detected on arm S"]
        P3["Agent suggests: all-red 60s + redirect"]
        P4["[✅ Approve] [❌ Reject] [✏️ Modify] ⏱️ 30s"]
    end
```

---

## 7. Sequence Diagram: Agent Decision Process

```mermaid
sequenceDiagram
    participant CAM as 📹 Camera
    participant MAPS as 🗺️ Maps API
    participant DET as 🚦 Congestion Detector
    participant LLM as 🤖 LLM Agent
    participant DASH as 📊 Dashboard
    participant SIM as 🖥️ Simulation

    loop Every 1 second
        CAM->>DET: Vehicle count, speed, queue
        MAPS->>DET: Speed per segment, incidents
        DET->>DET: Calculate congestion score
        DET->>LLM: {N: 45, S: 12, E: 30, W: 8} vehicles

        LLM->>LLM: Analyze + reason
        Note over LLM: "Hướng Bắc ùn tắt nặng,<br/>đề xuất kéo dài<br/>đèn xanh 20s"

        LLM->>LLM: Parse JSON decision
        LLM->>DASH: {action: "extend_green", severity: "warning"}

        alt Severity = Normal
            LLM->>SIM: Apply +20s green N
            SIM-->>DASH: Visualize updated signal
        else Severity = Critical
            LLM->>DASH: ⚠️ Request human approval
            DASH-->>CSGT: [Approve] [Reject] ⏱️ 30s
            CSGT->>DASH: Approve
            DASH->>SIM: Apply decision
        end
    end
```

---

## 8. A/B Test Evaluation Flow

```mermaid
flowchart TB
    START["🚀 Start Evaluation"]

    subgraph BASELINE["📊 Baseline Run (Agent OFF)"]
        B1["Fixed cycle: 30s green / 5s yellow"]
        B2["Run simulation 10 minutes"]
        B3["Record metrics:<br/>• avgWaitTime<br/>• queueLength<br/>• throughput"]
        B4["Save baseline results"]
    end

    subgraph AGENT_RUN["🤖 Agent Run (Agent ON)"]
        A1["Enable LLM agent"]
        A2["Run simulation 10 minutes"]
        A3["Record same metrics<br/>with agent active"]
        A4["Save agent results"]
    end

    subgraph COMPARE["📈 Compare Results"]
        C1["avgWaitTime: 65s → 48s<br/>↓26%"]
        C2["queueLength: 240m → 175m<br/>↓27%"]
        C3["throughput: 95 → 118 v/min<br/>↑24%"]
    end

    START --> BASELINE
    BASELINE --> AGENT_RUN
    AGENT_RUN --> COMPARE

    style BASELINE fill:#fee2e2,stroke:#dc2626
    style AGENT_RUN fill:#dbeafe,stroke:#2563eb
    style COMPARE fill:#dcfce7,stroke:#16a34a
```

---

## 9. Deployment Architecture

```mermaid
flowchart TB
    subgraph CLIENT["👤 Client Side"]
        BROWSER["🌐 Browser<br/>Next.js Dashboard"]
        PHONE["📱 Citizen App<br/>(Future)"]
    end

    subgraph EDGE["⚡ Edge Layer"]
        CAM["📹 Camera / RTSP Stream"]
        YOLO_EDGE["YOLO v8<br/>Edge Processing"]
    end

    subgraph CLOUD["☁️ Cloud (GCP)"]
        API["🛤️ Next.js API Routes"]
        LLM_SVC["🤖 LLM Service<br/>Groq / OpenRouter / Alibaba"]
        FIRESTORE["🔥 Firebase Firestore<br/>Real-time State"]
        STORAGE["💾 Cloud Storage<br/>Historical Logs"]
    end

    BROWSER -->|"Real-time WebSocket"| API
    PHONE -->|"Future API"| API
    CAM -->|"Video Stream"| YOLO_EDGE
    YOLO_EDGE -->|"Processed Data"| API
    API --> LLM_SVC
    API --> FIRESTORE
    API --> STORAGE
    FIRESTORE --> BROWSER
```

---

*Architecture diagrams: 2026-03-24*
*Project: TrafficFlow Agent — GDGoC Hackathon Vietnam 2026*
