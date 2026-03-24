# TrafficFlow Agent — System Architecture

> Mermaid diagrams for GDGoC Hackathon Vietnam 2026 | Stage 1: Proposal

---

## 1. High-Level System Architecture

```mermaid
flowchart TB
    subgraph INPUT["INPUT DATA LAYER"]
        CAMERA["Camera Streams (YOLO v8)"]
        MAPS["Google Maps Traffic API"]
        WEATHER["Weather API"]
        CALENDAR["Calendar API (Holiday/Events)"]
    end

    subgraph PROCESS["PROCESSING LAYER"]
        YOLO["YOLO Processor Count · Classify · Track"]
        MERGE["Data Fusion Camera + Maps + Context"]
        CONGESTION["Congestion Detector Density + Speed + Queue"]
    end

    subgraph AGENT["AI AGENT LAYER"]
        OBSERVE["OBSERVE Collect metrics"]
        REASON["REASON LLM Analysis"]
        DECIDE["DECIDE Action Selection"]
        ACT["ACT Apply Signal Change"]
        FEEDBACK["FEEDBACK Loop back"]
    end

    subgraph OUTPUT["OUTPUT LAYER"]
        SIM["Simulation Engine (Demo)"]
        SCATS["SCATS/ITMS API (Future)"]
        DASHBOARD["CSGT Dashboard Real-time Monitoring"]
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
    subgraph OBSERVE["OBSERVE"]
        O1["Camera: Count vehicles Speed per lane Queue length"]
        O2["Maps: Speed per segment Congestion color Incidents"]
        O3["Compute: Congestion score density·0.4 + (50-speed)·0.3 + queue·0.3"]
    end

    subgraph REASON["REASON (LLM)"]
        R1["Build prompt: Metrics + Goal + Thresholds"]
        R2["Call LLM: Groq → OpenRouter → Alibaba"]
        R3["Parse JSON: action + target + duration + reasoning"]
    end

    subgraph DECIDE["DECIDE"]
        D1{"Severity?"}
        D2["NORMAL Auto-apply"]
        D3["WARNING Dashboard alert"]
        D4["CRITICAL Human approval"]
    end

    subgraph ACT["ACT"]
        A1["Send signal command to simulation/SCATS"]
        A2["Update traffic light green/red duration"]
    end

    OBSERVE --> REASON
    REASON --> DECIDE
    D1 -->|"normal"| D2
    D1 -->|"warning"| D3
    D1 -->|"critical"| D4
    D2 --> ACT
    D3 --> DASHBOARD["Dashboard Notify CSGT"]
    D4 --> DASHBOARD
    DASHBOARD -->|"30s timeout or approve"| ACT
    ACT --> FEEDBACK["FEEDBACK Re-observe traffic state"]
    FEEDBACK --> OBSERVE
```

---

## 3. Data Flow Architecture

```mermaid
flowchart TB
    subgraph CAMERA_LAYER["CAMERA LAYER"]
        C1["Public Traffic Cameras RTSP/HTTP streams"]
        C2["Synthetic Data Mock · UA-DETRAC · BDD100K"]
    end

    subgraph PROCESSING["PROCESSING (Edge/Cloud)"]
        YOLO["YOLO v8 Object Detection"]
        TRACK["Object Tracking Vehicle Speed Estimation"]
        COUNT["Vehicle Counter Per-lane density"]
    end

    subgraph MAPS_LAYER["MAPS LAYER"]
        GMAPS["Google Maps Traffic API"]
        OSRM["OSRM (Backup)"]
    end

    subgraph FUSION["DATA FUSION"]
        F1["Merge Camera data (local detail)"]
        F2["Merge Maps data (global view)"]
        F3["Add Context: Time · Weather · Events"]
    end

    subgraph OUTPUT["OUTPUTS"]
        METRICS["Real-time Metrics Dashboard"]
        DECISION["Agent Decision Signal Control"]
        HISTORY["Historical Log A/B Evaluation"]
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
    START["Start Detection Loop Every 1 second"]
    INPUT["Get lane metrics: vehicleCount, avgSpeed, queueLength"]
    CONTEXT["Get context: timeOfDay, dayOfWeek, weather"]
    CALC["Calculate congestion score"]
    SCORE{"Score"}
    CLEAR["Status: CLEAR Green time: default"]
    MODERATE["Status: MODERATE Green time: +10s"]
    HEAVY["Status: HEAVY Green time: +20s Alert: WARNING"]
    GRIDLOCK["Status: GRIDLOCK Green time: +30s Alert: CRITICAL Request human approval"]

    START --> INPUT
    INPUT --> CONTEXT
    CONTEXT --> CALC
    CALC["Score = density × 0.4 + (50 - speed) × 0.3 + queue/10 × 0.3"]
    SCORE -->|"< 20"| CLEAR
    SCORE -->|"20-40"| MODERATE
    SCORE -->|"40-60"| HEAVY
    SCORE -->|"> 60"| GRIDLOCK
```

---

## 5. LLM Multi-Provider Architecture

```mermaid
flowchart TB
    PROMPT["Build Agent Prompt Metrics + Goal + JSON schema"]

    PROVIDERS["Try Providers in Order"]

    GROQ["Groq API llama-3.3-70b Fast · Cheap"]
    OPENROUTER["OpenRouter gpt-4o-mini / qwen Fallback"]
    ALIBABA["Alibaba Qwen qwen-turbo Last resort"]

    PARSE["Parse JSON Response: action, target, duration, severity"]
    VALID{"Valid JSON?"}
    RULE["Rule-Based Fallback Simple threshold logic"]
    OUTPUT["Agent Decision Ready"]

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
```

---

## 6. Dashboard Layout

```mermaid
flowchart TB
    subgraph HEADER["TrafficFlow Agent Dashboard"]
        H1["Agent Status: ONLINE"]
        H2["Mode: AUTO"]
        H3["Last Action: +20s green N"]
    end

    subgraph BODY["Main Content"]
        subgraph LEFT["Left Panel: Map View"]
            M1["Interactive Map Trường Chinh – Cách Mạng Tháng 8"]
            M2["Traffic overlay green/yellow/red"]
            M3["Signal indicators per arm (N/S/E/W)"]
            M4["Vehicle animation real-time movement"]
        end

        subgraph MIDDLE["Middle Panel: Live Metrics"]
            MET1["Avg Wait Time"]
            MET2["Queue Length"]
            MET3["Throughput"]
            MET4["Incidents: 0"]
        end

        subgraph RIGHT["Right Panel: Agent Status"]
            A1["Online"]
            A2["Mode: AUTO / PENDING_APPROVAL / MANUAL"]
            A3["LLM Provider: Groq"]
            A4["Last Decision: reason here..."]
        end
    end

    subgraph FOOTER["Bottom: Pending Approvals"]
        P1["Critical Alert"]
        P2["Accident detected on arm S"]
        P3["Agent suggests: all-red 60s + redirect"]
        P4["Approve | Reject | Modify 30s"]
    end
```

---

## 7. Sequence Diagram: Agent Decision Process

```mermaid
sequenceDiagram
    participant CAM as Camera
    participant MAPS as Maps API
    participant DET as Congestion Detector
    participant LLM as LLM Agent
    participant DASH as Dashboard
    participant SIM as Simulation

    loop Every 1 second
        CAM->>DET: Vehicle count, speed, queue
        MAPS->>DET: Speed per segment, incidents
        DET->>DET: Calculate congestion score
        DET->>LLM: {N: 45, S: 12, E: 30, W: 8} vehicles

        LLM->>LLM: Analyze + reason
        Note over LLM: "North arm is heavily congested, recommend extending green by 20s"

        LLM->>LLM: Parse JSON decision
        LLM->>DASH: {action: "extend_green", severity: "warning"}

        alt Severity = Normal
            LLM->>SIM: Apply +20s green N
            SIM-->>DASH: Visualize updated signal
        else Severity = Critical
            LLM->>DASH: Request human approval
            DASH-->>CSGT: Approve / Reject 30s
            CSGT->>DASH: Approve
            DASH->>SIM: Apply decision
        end
    end
```

---

## 8. A/B Test Evaluation Flow

```mermaid
flowchart TB
    START["Start Evaluation"]

    subgraph BASELINE["Baseline Run (Agent OFF)"]
        B1["Fixed cycle: 30s green / 5s yellow"]
        B2["Run simulation 10 minutes"]
        B3["Record metrics: avgWaitTime, queueLength, throughput"]
        B4["Save baseline results"]
    end

    subgraph AGENT_RUN["Agent Run (Agent ON)"]
        A1["Enable LLM agent"]
        A2["Run simulation 10 minutes"]
        A3["Record same metrics with agent active"]
        A4["Save agent results"]
    end

    subgraph COMPARE["Compare Results"]
        C1["avgWaitTime: 65s → 48s improvement"]
        C2["queueLength: 240m → 175m improvement"]
        C3["throughput: 95 → 118 veh/min improvement"]
    end

    START --> BASELINE
    BASELINE --> AGENT_RUN
    AGENT_RUN --> COMPARE
```

---

## 9. Deployment Architecture

```mermaid
flowchart TB
    subgraph CLIENT["Client Side"]
        BROWSER["Browser Next.js Dashboard"]
        PHONE["Citizen App (Future)"]
    end

    subgraph EDGE["Edge Layer"]
        CAM["Camera / RTSP Stream"]
        YOLO_EDGE["YOLO v8 Edge Processing"]
    end

    subgraph CLOUD["Cloud (GCP)"]
        API["Next.js API Routes"]
        LLM_SVC["LLM Service Groq / OpenRouter / Alibaba"]
        FIRESTORE["Firebase Firestore Real-time State"]
        STORAGE["Cloud Storage Historical Logs"]
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
