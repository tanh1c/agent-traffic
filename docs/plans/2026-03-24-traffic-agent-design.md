# Traffic Agent System — Design Document
**GDGoC Hackathon Vietnam 2026 | Stage 1: Proposal**

---

## 1. Project Overview

**Project Name:** TrafficFlow Agent
**Theme:** "Agents of Change" — Agentic AI cho điều phối giao thông

### Elevator Pitch
Mỗi ngày, hàng triệu người dân TP.HCM mắc kẹt trong ùn tắc tại các ngã tư như Trường Chinh – Cách Mạng Tháng 8, lãng phí hàng giờ chờ đợi dưới nắng mưa. Cảnh sát giao thông kiệt sức điều phối thủ công, trong khi hệ thống đèn tín hiệu vẫn chạy theo lịch cố định không phù hợp thực tế.

TrafficFlow Agent là hệ thống AI tự chủ (autonomous AI Agent) sử dụng camera real-time và dữ liệu Google Maps để phát hiện ùn tắc theo phương pháp hybrid (density + speed + queue length), suy luận qua LLM đa nhà cung cấp (Groq / OpenRouter / Alibaba Qwen) để đưa ra quyết định điều chỉnh đèn tín hiệu — giảm thời gian chờ trung bình, giảm phụ thuộc vào điều phối thủ công.

### Why Agentic AI?
| Đặc điểm Agentic | Cách thể hiện trong hệ thống |
|---|---|
| **Goal-driven** | Agent luôn đo lường: thời gian chờ trung bình, độ dài hàng đợi |
| **Planning** | LLM reasoning chain phân tích tình huống → đề xuất chiến lược điều chỉnh đèn |
| **Autonomy** | Tự quyết định & apply cho case thường; chỉ hỏi human khi nghiêm trọng |
| **Feedback Loop** | Sau mỗi action → observe lại → adjust liên tục |

---

## 2. Problem Statement

- **Ai bị ảnh hưởng:** Hàng triệu người dân TP.HCM, đặc biệt khu vực Trường Chinh – Cách Mạng Tháng 8
- **Tại sao quan trọng:** Ùn tắt gây thiệt hại kinh tế (thời gian, nhiên liệu), ô nhiễm môi trường, stress cho người tham gia giao thông
- **Giải pháp hiện tại còn thiếu gì:**
  - Đèn tín hiệu chạy theo lịch cố định, không thích ứng real-time
  - Không có hệ thống tự động phát hiện + phản ứng với ùn tắt
  - Cảnh sát giao thông phải điều phối thủ công, tốn nhân lực

---

## 3. System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    TRAFFIC AGENT SYSTEM                     │
│                                                             │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────────┐  │
│  │ Camera   │    │ Traffic     │    │ Google Maps      │  │
│  │ Streams  │───▶│ Processor   │───▶│ Traffic Data     │  │
│  │ (YOLO)   │    │ (density,   │    │ (speed, flow)    │  │
│  └──────────┘    │  speed)     │    └──────────────────┘  │
│                  └──────┬───────┘           │               │
│                         │                   │               │
│                         ▼                   ▼               │
│                  ┌──────────────────────────────┐           │
│                  │      AI AGENT               │              │
│                  │  ┌──────────────────────┐   │              │
│                  │  │ Congestion Detector  │   │              │
│                  │  └─────────┬────────────┘   │              │
│                  │            │                 │              │
│                  │  ┌─────────▼────────────┐   │              │
│                  │  │ LLM Reasoner         │   │              │
│                  │  │ (multi-provider)      │   │              │
│                  │  └─────────┬────────────┘   │              │
│                  │            │                 │              │
│                  │  ┌─────────▼────────────┐   │              │
│                  │  │ Decision Engine      │   │              │
│                  │  │ + Action Planner     │   │              │
│                  │  └──────────────────────┘   │              │
│                  └─────────────────────────────┘              │
│                               │                               │
│            ┌───────────────────┼──────────────────┐           │
│            ▼                   ▼                   ▼           │
│   ┌─────────────┐    ┌─────────────────┐  ┌─────────────┐   │
│   │ Simulation  │    │ SCATS/ITMS API  │  │  Dashboard   │   │
│   │ Engine      │    │ (future-ready)   │  │  CSGT        │   │
│   │ (demo)      │    │                 │  │  (monitor)   │   │
│   └─────────────┘    └─────────────────┘  └─────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Congestion Detection Method: Hybrid
- **Density** (số xe trên một đơn vị diện tích/làn) — từ YOLO
- **Speed** (tốc độ di chuyển trung bình) — từ Google Maps + YOLO tracking
- **Queue length** (độ dài hàng đợi tính bằng mét) — từ YOLO bounding boxes
- Kết hợp cả ba yếu tố + thời gian trong ngày + ngày trong tuần

---

## 4. Agent Decision Flow

```
Step 1: OBSERVE
  Camera (YOLO) + Google Maps → Metrics
  Density: N xe/làn | Speed: V km/h | Queue: L mét

Step 2: REASON (LLM)
  Prompt: "[Trạng thái hiện tại] + [Mục tiêu] + [Ngưỡng]"
  → LLM đề xuất: điều chỉnh đèn nào, thêm/bớt bao lâu, giải thích

Step 3: DECIDE
  Tình huống thường     → Auto-apply (agent tự quyết định)
  Tình huống nghiêm trọng → Dashboard CSGT approve trong 30s
    (tai nạn, sự kiện lớn, tắc nặng kéo dài)
    → Auto-apply sau 30s nếu không ai phản hồi

Step 4: ACT + FEEDBACK
  Command → Simulation Engine → Đèn chuyển → Camera observe → Loop lại
```

### Multi-Provider LLM
- **Primary:** Groq (nhanh, rẻ, API ổn định)
- **Fallback:** OpenRouter, Alibaba Qwen
- Agent chọn provider phù hợp với tình huống (simple → rẻ, complex → mạnh)

---

## 5. Data Sources

| Nguồn | Dữ liệu lấy ra | Mục đích |
|---|---|---|
| **YOLO v8** (Camera) | Số xe, loại xe, tốc độ, queue length | Chi tiết cục bộ từng làn |
| **Google Maps Traffic API** | Speed per segment, congestion color, incident | Tầm nhìn tổng thể toàn tuyến |
| **OSRM** (backup) | Routing engine | Backup cho Google Maps |
| **Weather API** | Trời mưa → baseline điều chỉnh | Contextual awareness |
| **Public holiday calendar** | Tết, cuối tuần → pattern khác | Historical baseline |

### Camera Strategy
- **Public traffic cameras** (nếu có online stream): RTSP/HTTP stream
- **Synthetic / mock data** (khi không có camera thật): Tự tạo video giả lập hoặc dùng dataset (UA-DETRAC, BDD100K)
- Cả hai cùng chạy → đảm bảo demo không bị lỗi

### So sánh với Google Maps
> Google Maps dùng GPS probe data (điện thoại người dùng ẩn danh) → thấy "xe đang chậm" nhưng không biết **tại sao**, không phân biệt được từng làn. Hệ thống của chúng tôi dùng camera → thấy chi tiết từng làn, từng loại phương tiện, phát hiện tai nạn/vật cản trực tiếp.

---

## 6. Simulation & Demo

### Demo chính: Simulation Engine
- Bản đồ khu vực Trường Chinh – Cách Mạng Tháng 8 (TP.HCM)
- Animation luồng xe di chuyển
- Đèn xanh/đỏ chuyển theo agent decision
- Real-time visualization trên web dashboard

### A/B Test Evaluation
- **Scenario:** Giờ cao điểm (17:00–19:00) tại khu vực demo
- **Metric chính:**
  - Thời gian chờ trung bình (Average wait time)
  - Số lần dừng trung bình (Average stops)
  - Độ dài hàng đợi (Queue length)
  - Throughput (xe/phút qua ngã tư)
- **So sánh:** Agent OFF (baseline cố định) vs Agent ON → trình bày số liệu cụ thể

---

## 7. Dashboard

```
┌──────────────────────────────────────────────────────────────┐
│  🗺️ MAP VIEW          │  AGENT STATUS      │  LIVE METRICS  │
│  (Interactive map)    │  ● Online          │  Avg wait: 45s │
│  Camera dots +        │  Mode: AUTO        │  Queue: 180m   │
│  traffic flow +       │  Last action:     │  Throughput:   │
│  signal status        │  +20s green N     │  120 veh/min   │
├──────────────────────────────────────────────────────────────┤
│  📋 PENDING APPROVAL (nếu có case nghiêm trọng)              │
│  ⚠️ [Mô tả tình huống]                                       │
│     Agent đề xuất: [hành động]                                │
│     [Approve] [Reject] [Modify] ⏱️ 30s auto-apply             │
└──────────────────────────────────────────────────────────────┘
```

- **Map view:** Bản đồ interactive thể hiện luồng xe, mật độ theo màu
- **Agent status:** Trạng thái online/offline, auto/manual
- **Live metrics:** Các số đo real-time
- **Pending approval:** Human-in-the-loop cho case nghiêm trọng

---

## 8. Technology Stack

| Layer | Technology |
|---|---|
| **LLM** | Multi-provider: Groq / OpenRouter / Alibaba Qwen (API) |
| **Agent Framework** | LangChain / LangGraph (reasoning chain) |
| **Object Detection** | YOLO v8 (vehicle counting, classification, tracking) |
| **Map Data** | Google Maps Traffic API + OSRM (backup) |
| **Simulation** | Custom engine (web-based visualization) |
| **Future Real-world** | SCATS/ITMS API integration |
| **Frontend** | Next.js / React (dashboard) |
| **Cloud** | Google Cloud Platform (align với GDG) |
| **Database** | Firebase Firestore (real-time state) |

---

## 9. Extensions (Future Work)

### B) Phương tiện ưu tiên
- Phát hiện xe buýt, xe cấp cứu qua YOLO → agent ưu tiên kéo dài đèn cho làn đó
- Social impact rõ ràng

### C) Dự báo ùn tắt trước khi xảy ra
- Dùng historical pattern + current state → predict tắc đường trước 15-30 phút
- Agent chủ động điều chỉnh đèn **trước** khi tắt thay vì phản ứng

### D) Dashboard cho cảnh sát giao thông
- Dashboard giám sát thời gian thực
- Human-in-the-loop cho decision nghiêm trọng
- Notification khi hệ thống cần can thiệp

---

## 10. Success Metrics

| Metric | Baseline (Agent OFF) | Target (Agent ON) |
|---|---|---|
| Thời gian chờ TB | Baseline cố định | Giảm 15–25% |
| Số lần dừng TB | Baseline cố định | Giảm 10–20% |
| Queue length | Baseline cố định | Giảm 15–30% |
| Throughput | Baseline cố định | Tăng 10–20% |

Metrics được đo qua A/B test trong simulation.

---

## 11. Development Plan (Hackathon)

### Day 1
- Thiết lập simulation engine (bản đồ + animation xe)
- Kết nối YOLO cho object detection
- Kết nối Google Maps Traffic API
- Demo flow cơ bản: camera → detect → detect congestion → show metrics

### Day 2
- Tích hợp LLM agent reasoning
- Hoàn thiện decision loop (Observe → Reason → Decide → Act)
- Xây dashboard cho CSGT
- A/B test evaluation run

### Demo
- Live demo: agent điều chỉnh đèn real-time trên simulation
- A/B test results trình bày với ban giám khảo
- Slide deck theo proposal template

---

*Document created: 2026-03-24*
*Stage: Brainstorming completed → Next: Implementation Plan*
