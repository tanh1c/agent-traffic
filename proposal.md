# GDGoC Hackathon Vietnam 2026 - Project Proposal

---

## 🧑‍💻 Team Information

**Team Name:**
[TÊN ĐỘI - CẦN CẬP NHẬT]

| STT | Họ và tên | Vai trò |
|-----|----------|--------|
| 1   | [Họ và tên - CẦN CẬP NHẬT] | [Vai trò - CẦN CẬP NHẬT] |
| 2   | [Họ và tên - CẦU CẬP NHẬT] | [Vai trò - CẦN CẬP NHẬT] |
| 3   | [Họ và tên - CẦN CẬP NHẬT] | [Vai trò - CẦN CẬP NHẬT] |
| 4   | [Họ và tên - CẦN CẬP NHẬT] | [Vai trò - CẦN CẬP NHẬT] |

---

## 1. 🚀 Project Overview

### 1.1 Project Name

**TrafficFlow Agent** — Hệ thống Agentic AI Tự động Điều phối Giao thông Thông minh

---

### 1.2 Elevator Pitch (4–6 câu)

- **Vấn đề:** Mỗi ngày, hàng triệu người dân TP.HCM mắc kẹt trong ùn tắc tại các ngã tư như Trường Chinh – Cách Mạng Tháng 8, lãng phí hàng giờ chờ đợi dưới nắng mưa. Hệ thống đèn tín hiệu chạy theo lịch cố định không thích ứng với thực tế, trong khi cảnh sát giao thông phải điều phối thủ công trong điều kiện khắc nghiệt.

- **Giải pháp:** TrafficFlow Agent là hệ thống AI tự chủ sử dụng camera real-time và dữ liệu Google Maps để phát hiện ùn tắt theo phương pháp hybrid (mật độ + tốc độ + độ dài hàng đợi), suy luận qua LLM đa nhà cung cấp (Groq / OpenRouter / Alibaba Qwen) để đưa ra quyết định điều chỉnh đèn tín hiệu tối ưu.

- **Đối tượng:** Người tham gia giao thông tại TP.HCM, đặc biệt khu vực đông đúc; cảnh sát giao thông (CSGT); cơ quan quản lý giao thông đô thị.

- **Giá trị mang lại:** Giảm thời gian chờ trung bình 15–25%, giảm độ dài hàng đợi 15–30%, tăng throughput 10–20%, giảm phụ thuộc vào điều phối thủ công của CSGT, giảm ô nhiễm môi trường do kẹt xe kéo dài.

---

### 1.3 Why Agentic AI?

Hệ thống của bạn thể hiện tính Agentic như thế nào?

- 🎯 **Goal:** Agent luôn có mục tiêu rõ ràng — giảm thời gian chờ trung bình và tránh gridlock. Mọi hành động đều được đo lường và đánh giá theo KPI cụ thể.

- 🧠 **Planning:** Agent dùng LLM (Groq/OpenRouter/Alibaba Qwen) để phân tích tình hình giao thông đa chiều (density, speed, queue, thời gian, ngày trong tuần), đề xuất chiến lược điều chỉnh đèn với reasoning chain rõ ràng — không chỉ là rule-based đơn giản.

- 🤖 **Autonomy:** Agent tự quyết định và tự điều chỉnh cho các tình huống thông thường mà không cần chờ lệnh từ con người. Chỉ hỏi human approval khi phát hiện tình huống nghiêm trọng (tai nạn, sự kiện đặc biệt).

- 🔁 **Feedback Loop:** Sau mỗi hành động điều chỉnh đèn, agent liên tục observe lại trạng thái giao thông mới qua camera và Google Maps → re-analyze → re-decide. Cycle liên tục để đảm bảo hệ thống luôn thích ứng với thực tế.

---

## 2. 🧩 Problem & Solution

### 2.1 Problem Statement

- **Ai bị ảnh hưởng?**
  - Hàng triệu người dân TP.HCM — đặc biệt khu vực Trường Chinh, Cách Mạng Tháng 8, quận Tân Bình, quận 3
  - Tài xế xe buýt, tài xế xe công nghệ (Grab, Be, Gojek)
  - Cảnh sát giao thông (CSGT) phải điều phối thủ công giờ cao điểm
  - Doanh nghiệp logistics chịu thiệt hại do chậm trễ vận chuyển

- **Tại sao quan trọng?**
  - TP.HCM có > 8 triệu xe máy, > 800.000 ô tô (theo số liệu Sở GTVT)
  - Ùn tắt gây thiệt hại kinh tế ước tính 1.2–1.5% GDP mỗi năm
  - Khí thải từ xe chờ ùn tắt gây ô nhiễm, ảnh hưởng sức khỏe
  - CSGT làm việc quá tải trong điều kiện thời tiết khắc nghiệt

- **Giải pháp hiện tại còn thiếu gì?**
  - Đèn tín hiệu tại hầu hết các ngã tư chạy theo lịch cố định, không thích ứng với mật độ thực tế
  - Không có hệ thống tự động phát hiện + phản ứng nhanh với ùn tắt theo real-time
  - Google Maps chỉ thấy "đoạn đường đỏ" (GPS probe data) nhưng không biết nguyên nhân và không điều khiển được đèn
  - CSGT phải có mặt trực tiếp hoặc được gọi đến khi tình huống xấu đi

---

### 2.2 Key Questions

- **Làm sao AI có thể tự giải quyết bài toán giao thông?**
  → AI phân tích dữ liệu từ camera (YOLO) + Google Maps theo thời gian thực, đánh giá mức độ ùn tắt, và đưa ra quyết định điều chỉnh đèn dựa trên reasoning — không chỉ rule cố định.

- **Làm sao AI dùng nhiều công cụ (tools)?**
  → Agent dùng đồng thời: (1) YOLO để detect và đếm xe, (2) Google Maps API để lấy speed data, (3) LLM để reason và đề xuất action, (4) Simulation engine để visualize kết quả, (5) Dashboard để human-in-the-loop khi cần.

- **Làm sao đảm bảo độ tin cậy của hệ thống?**
  → Human-in-the-loop: agent chỉ tự động apply cho case thường; case nghiêm trọng (tai nạn, gridlock) luôn chờ approval từ CSGT trước khi hành động. A/B test evaluation đo lường hiệu quả thực tế qua simulation. Multi-provider LLM fallback đảm bảo hệ thống không bị down khi một provider lỗi.

---

### 2.3 Proposed Solution

#### System Overview

TrafficFlow Agent là hệ thống hybrid gồm 3 lớp dữ liệu:

```
[Camera Real-time (YOLO)]          [Google Maps Traffic API]
        │                                   │
        ▼                                   ▼
  Chi tiết cục bộ:                  Tầm nhìn tổng thể:
  - Số xe từng làn                  - Speed per segment
  - Loại phương tiện                 - Congestion color
  - Queue length cụ thể              - Incident reports
        │                                   │
        └──────────────┬────────────────────┘
                       ▼
              [AI AGENT (LLM)]
              - Congestion Detection
              - Reasoning & Planning
              - Decision Engine
                       │
              ┌────────┴────────┐
              ▼                ▼
    [Simulation Engine]  [SCATS/ITMS API]
         (Demo)           (Future-ready)
              │
              ▼
        [Dashboard CSGT]
        (Real-time monitoring
         + Human approval)
```

#### Agent Workflow

1. **Input:** Camera frames (YOLO) + Google Maps data → ghép thành bức tranh toàn diện
2. **Analysis:** Congestion Detector phân tích density + speed + queue length → phân loại mức độ ùn tắt
3. **Planning:** LLM đa nhà cung cấp suy luận tình huống, đề xuất chiến lược điều chỉnh đèn
4. **Action:** Apply quyết định vào simulation (demo) hoặc SCATS/ITMS (thực tế)
5. **Feedback:** Quan sát kết quả → loop lại bước 1

#### Key Features

- **Hybrid Congestion Detection:** Kết hợp camera (YOLO) + Google Maps để phát hiện ùn tắt chính xác hơn bất kỳ giải pháp đơn lẻ nào. Camera thấy chi tiết từng làn; Maps thấy bức tranh toàn tuyến.
- **LLM-Powered Reasoning Agent:** Agent dùng LLM (Groq / OpenRouter / Alibaba Qwen) để reason theo chuỗi, đề xuất action cụ thể kèm giải thích. Multi-provider fallback đảm bảo độ sẵn sàng.
- **Human-in-the-Loop Dashboard:** CSGT giám sát real-time trên dashboard, nhận notification khi agent cần approval cho case nghiêm trọng. Quyết định được apply sau 30s nếu không ai phản hồi.

#### Unique Selling Point (USP)

| So sánh | Google Maps Traffic | TrafficFlow Agent |
|---------|---------------------|-------------------|
| Dữ liệu | GPS probe (điện thoại ẩn danh) | Camera real-time + GPS |
| Độ chi tiết | Toàn tuyến, không phân biệt làn | Từng làn, từng loại xe |
| Phát hiện sự cố | Không (chỉ thấy đỏ) | Phát hiện tai nạn, vật cản |
| Điều khiển đèn | Không | Có (qua SCATS/ITMS API) |
| Tính Agentic | Không | Có (reasoning, planning, autonomy) |

→ **Điểm khác biệt cốt lõi:** Camera thấy được "tại sao" đoạn đường đỏ (tai nạn? làn nào tắt? xe gì?) trong khi Google Maps chỉ thấy "đoạn đường đỏ" mà không biết nguyên nhân.

---

## 3. ⚙️ Technology Stack

### 3.1 AI & Machine Learning

- **LLM:** Groq API / OpenRouter / Alibaba Qwen (multi-provider, flexible — dùng nhiều nguồn để đảm bảo availability + tối ưu chi phí)
- **Agent Framework:** LangChain / LangGraph (để xây reasoning chain rõ ràng, trace được)
- **Object Detection:** YOLO v8 (vehicle counting, classification, tracking) — xử lý real-time trên edge hoặc cloud
- **Embedding:** Không cần (dùng LLM trực tiếp cho reasoning)

### 3.2 Infrastructure

- **Cloud:** Google Cloud Platform (align với GDG — có thể dùng Google Cloud credits từ hackathon)
- **Backend:** Next.js API routes (frontend + backend cùng stack, đơn giản hóa deployment)
- **Deployment:** Vercel / Google Cloud Run (fast, free tier đủ cho demo)

### 3.3 Development

- **Frontend:** Next.js 14+ (App Router) + React + Tailwind CSS
- **Database:** Firebase Firestore (real-time state cho dashboard)
- **Simulation Engine:** Custom web-based (Canvas/SVG cho visualization)
- **Camera Processing:** YOLO v8 (via Python backend hoặc ONNX runtime on browser)

### 3.4 Competitive Advantage

- **Điểm mạnh công nghệ:**
  - Kết hợp 3 nguồn dữ liệu: Camera + Maps + Weather (contextual)
  - Multi-provider LLM: không phụ thuộc một nhà cung cấp duy nhất
  - YOLO real-time detection: xử lý video stream trực tiếp, không cần gửi raw video
- **Khả năng mở rộng:**
  - Thêm ngã tư mới: chỉ cần thêm camera + update config
  - Mở rộng features: ưu tiên xe cấp cứu, dự báo tắt, tích hợp thêm Maps API
  - Scale lên toàn thành phố: dùng distributed agent coordination
- **Tính độc đáo:**
  - Chưa có giải pháp tại Việt Nam kết hợp camera + LLM agent + Maps cho điều phối đèn tín hiệu
  - Human-in-the-loop design an toàn và thực tế cho context Việt Nam

---

## 4. 👥 Target Users

- **Cảnh sát giao thông (CSGT):**
  - Dashboard giám sát thời gian thực → giảm áp lực điều phối thủ công
  - Nhận notification khi cần can thiệp → tập trung vào tình huống thực sự nghiêm trọng

- **Cơ quan quản lý giao thông đô thị:**
  - Dashboard tổng quan cho nhiều ngã tư
  - Báo cáo A/B test → đánh giá hiệu quả các phương án điều chỉnh

- **Người dân tham gia giao thông:**
  - Giảm thời gian chờ, giảm stress khi di chuyển
  - (Future) Nhận thông báo từ app về tình trạng giao thông phía trước

- **Use case cụ thể:**
  - Giờ cao điểm 7:00–9:00 và 17:00–19:00 tại Trường Chinh – Cách Mạng Tháng 8
  - Sự kiện đặc biệt (concert, giải bóng đá) gần khu vực
  - Thời tiết xấu (mưa) → baseline tự điều chỉnh

---

## 5. 📈 Feasibility

### 5.1 Development Plan

#### During Hackathon

**Day 1 — Foundation & Core Demo:**

| Thời gian | Nhiệm vụ |
|-----------|---------|
| Sáng (9:00–12:00) | Setup Next.js project, folder structure, TypeScript types |
| Trưa (12:00–13:00) | Demo simulation engine (bản đồ + xe di chuyển baseline) |
| Chiều (13:00–18:00) | Tích hợp mock YOLO (vehicle generator + congestion detector) |
| Tối (18:00–22:00) | Kết nối Google Maps Traffic API (hoặc mock fallback) |
| Đêm (22:00–24:00) | Baseline simulation chạy → record metrics (Agent OFF) |

**Day 2 — Agent & Polish:**

| Thời gian | Nhiệm vụ |
|-----------|---------|
| Sáng (9:00–12:00) | Tích hợp LLM agent (multi-provider + reasoning chain) |
| Trưa (12:00–13:00) | Hoàn thiện Dashboard (map view + metrics + agent status) |
| Chiều (13:00–17:00) | A/B test: chạy Agent ON → generate comparison report |
| Chiều muộn (17:00–19:00) | Hoàn thiện slide deck |
| Tối (19:00–22:00) | Full demo run-through, fix bugs, prepare submission |

**Demo (Pitching):**
- Live demo: Dashboard real-time → agent phát hiện ùn tắt → LLM reasoning hiện lên → đèn điều chỉnh → metrics cải thiện
- A/B test results: số liệu cụ thể trước/sau khi có agent
- Slide deck: 10 slides theo proposal template

#### Post-hackathon

| Giai đoạn | Mục tiêu |
|-----------|---------|
| MVP (1–2 tháng) | Thêm public traffic cameras, tích hợp SCATS API thực tế |
| Beta (3–4 tháng) | Pilot tại 3–5 ngã tư ở TP.HCM, thu thập feedback |
| Scale (6+ tháng) | Mở rộng toàn thành phố, tích hợp phương tiện ưu tiên |

---

### 5.2 Budget

| Hạng mục | Chi phí ước tính (USD) |
|---------|----------------------|
| Google Cloud Platform | ~$0–$50 (credits từ hackathon GCP) |
| API Calls (LLM - Groq/OpenRouter) | ~$5–$20 (tuỳ usage) |
| Google Maps API | ~$0–$50 (free tier đủ cho demo) |
| Domain + Hosting (Vercel) | $0 (free tier) |
| Tools & Libraries | $0 (tất cả open source) |
| **Total ước tính** | **~$5–$120** |

---

## 6. 📊 Impact

### KPI đo hiệu quả:

| Chỉ số | Baseline (Agent OFF) | Target (Agent ON) | Mục tiêu |
|--------|---------------------|-------------------|---------|
| Thời gian chờ trung bình | 60–90s (ước tính) | 45–68s | ↓ 15–25% |
| Độ dài hàng đợi trung bình | 200–300m | 140–255m | ↓ 15–30% |
| Số lần dừng trung bình | 3–5 lần/xe | 2–4 lần/xe | ↓ 10–20% |
| Throughput (xe/phút) | 80–100 | 90–120 | ↑ 10–20% |
| Số vụ gridlock/ngày | 2–3 | 0–1 | ↓ 50–70% |

*Các con số baseline dựa trên simulation với cấu hình đèn cố định tại Trường Chinh – Cách Mạng Tháng 8.*

### Giá trị mang lại:

- **Cá nhân:** Giảm thời gian di chuyển 10–20 phút/ngày cho người đi lại qua khu vực; giảm stress, tiết kiệm nhiên liệu
- **Tổ chức:** CSGT giảm áp lực điều phối thủ công, tập trung vào tình huống nghiêm trọng; công ty logistics giảm chi phí vận chuyển
- **Xã hội:** Giảm khí thải từ xe ùn tắt (ước tính giảm 5–10% CO2 tại khu vực); cải thiện chất lượng không khí đô thị; nâng cao hình ảnh TP.HCM hiện đại, ứng dụng công nghệ AI

---

## 7. 📚 References

- **LangChain Documentation:** https://docs.langchain.com/
- **YOLO v8 (Ultralytics):** https://docs.ultralytics.com/
- **Google Maps Traffic API:** https://developers.google.com/maps/documentation/traffic
- **Groq API:** https://console.groq.com/docs
- **OpenRouter:** https://openrouter.ai/docs
- **Alibaba Qwen API:** https://qwenlm.github.io/
- **SCATS Traffic System:** https://en.wikipedia.org/wiki/SCATS
- **GDGoC Hackathon Vietnam 2026:** https://gdg.community.dev/gdgoc-hackathon-vietnam-2026/

---

## 8. 📞 Contact

- **Email:** [EMAIL - CẦN CẬP NHẬT]
- **Phone:** [SỐ ĐIỆN THOẠI - CẦN CẬP NHẬT]

---

*Proposal created: 2026-03-24*
*Project: TrafficFlow Agent — GDGoC Hackathon Vietnam 2026*
*Theme: "Agents of Change" — Agentic AI for Smart Traffic Management*
