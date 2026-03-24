# TrafficFlow Agent

> Agentic AI System for Smart Traffic Signal Control — GDGoC Hackathon Vietnam 2026

## Overview

**TrafficFlow Agent** là hệ thống AI tự chủ (autonomous AI Agent) sử dụng camera real-time và dữ liệu Google Maps để phát hiện ùn tắt, suy luận qua LLM đa nhà cung cấp để đưa ra quyết định điều chỉnh đèn tín hiệu tối ưu tại các ngã tư đông đúc ở TP.HCM (Trường Chinh – Cách Mạng Tháng 8).

**Theme:** "Agents of Change" — Agentic AI cho Smart Traffic Management

## Project Structure

```
agent-traffic/
├── docs/
│   ├── proposal_template.md          # Template proposal (reference)
│   ├── proposal.md                   # Final proposal for Stage 1
│   └── plans/
│       ├── 2026-03-24-traffic-agent-design.md
│       └── 2026-03-24-traffic-agent-implementation-plan.md
├── traffic-sim/                      # Next.js simulation & dashboard
│   ├── components/
│   ├── lib/
│   ├── app/
│   └── ...
├── hackathon.md                      # Hackathon rules & criteria
└── README.md
```

## Tech Stack

| Layer | Technology |
|---|---|
| **AI Agent** | Groq / OpenRouter / Alibaba Qwen (multi-provider LLM) |
| **Agent Framework** | LangChain / LangGraph |
| **Object Detection** | YOLO v8 |
| **Map Data** | Google Maps Traffic API + OSRM |
| **Frontend** | Next.js 14+ (App Router) + React + Tailwind CSS |
| **Simulation** | Custom web engine (Canvas/SVG) |
| **Cloud** | Google Cloud Platform |

## Agent Architecture

```
Camera (YOLO) + Google Maps → Congestion Detector → LLM Reasoner → Decision Engine
                                                                      │
                                              ┌──────────┬──────────────┘
                                              ▼          ▼
                                       Simulation     SCATS API
                                         (Demo)    (Future-ready)
```

## Key Features

- **Hybrid Congestion Detection** — Camera (YOLO) + Google Maps cho độ chính xác cao nhất
- **LLM-Powered Reasoning** — Multi-provider LLM với reasoning chain rõ ràng
- **Human-in-the-Loop** — CSGT dashboard với approval cho case nghiêm trọng
- **A/B Test Evaluation** — So sánh baseline vs agent để đo lường hiệu quả

## Hackathon Stage

- [x] **Stage 1: Proposal** — `docs/proposal.md`
- [ ] **Stage 2: Prototype** — `traffic-sim/`
- [ ] **Stage 3: Final Demo**

## Team

> [!IMPORTANT]
> Thông tin team đang được cập nhật. Xem `docs/proposal.md` để điền.

## References

- [GDGoC Hackathon Vietnam 2026](https://gdg.community.dev/gdgoc-hackathon-vietnam-2026/)
- [LangChain Documentation](https://docs.langchain.com/)
- [YOLO v8](https://docs.ultralytics.com/)
- [Google Maps Traffic API](https://developers.google.com/maps/documentation/traffic)
