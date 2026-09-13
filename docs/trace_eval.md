# 📊 BÁO CÁO THU HOẠCH NGHIỆM THU BÀI LAB 3 (BƯỚC 3 — SUBMISSION ARTIFACT)

> **Họ và Tên Học viên:** Phạm Long Nhật  
> **Mã Sinh Viên / Mã Học viên:** 2A202602844  
> **Chủ đề Lựa chọn:** Gợi ý 1.1 — Trợ lý Học vụ & Tra cứu Lịch thi VinUni (tra cứu hồ sơ học vụ + đặt lịch tư vấn với Cố vấn học tập)  

---

## 1. BẢNG CHẤM ĐIỂM AGENTIC FIT SCORING MATRIX (ĐÁNH GIÁ CHỦ ĐỀ)

| Tiêu chí Đánh giá | Mức độ (1 - 5) | Giải trình chi tiết lý do chọn điểm |
| :--- | :---: | :--- |
| **1. Multi-step Reasoning** | 4 / 5 | Yêu cầu như "đặt lịch với cố vấn của tôi" phải tách thành chuỗi nối tiếp: tra cứu hồ sơ (`academic_query`) để biết cố vấn là ai → đặt lịch (`schedule_appointment`) → tổng hợp xác nhận. Không đạt 5 vì chuỗi suy luận ngắn (2–3 bước), chưa cần lập kế hoạch phức tạp. |
| **2. Tool Interaction** | 5 / 5 | GPA, trạng thái học, cố vấn và lịch hẹn là dữ liệu cá nhân, thay đổi theo thời gian, nằm trong CSDL học vụ — LLM không thể tự biết. Bắt buộc gọi Tool qua MCP Server; không có Tool thì Agent chỉ có thể từ chối hoặc bịa đặt (hallucination). |
| **3. Dynamic Decision** | 4 / 5 | Bước sau phụ thuộc trực tiếp vào Observation: `NOT_FOUND` → dừng, thông báo lịch sự, không đặt lịch; `SUCCESS` → lấy đúng `advisor` trong kết quả để truyền vào tham số đặt lịch. Không đạt 5 vì chỉ có 2 tool nên không gian nhánh quyết định còn hẹp. |
| **4. Long Horizon Goal** | 3 / 5 | Agent phải giữ mục tiêu "đặt được lịch hẹn" xuyên suốt nhiều bước ReAct trong một yêu cầu. Tuy nhiên chưa có memory giữa các phiên hay mục tiêu kéo dài nhiều ngày (thuộc Cấp 4 — Autonomous Agent), nên chấm mức trung bình. |
| **TỔNG ĐIỂM AGENTIC FIT** | **16 / 20** | *Nếu tổng điểm > 12/20: Bài toán rất phù hợp triển khai Agentic System.* → **16/20 > 12: Phù hợp triển khai ReAct Agent.** |

> **Đối chiếu sơ đồ quyết định:** Không phải mọi câu hỏi trong đề tài đều cần Agent. Câu hỏi quy chế chung (TC01) không cần Multi-step / Tool use / Dynamic Decision → Chatbot Baseline là đủ. Các yêu cầu tra cứu hồ sơ, đặt lịch và chuỗi "tra cứu → đặt lịch" (TC02–TC05) mới cần nâng cấp lên ReAct Agent System.

---

## 2. TRÍCH XUẤT KẾT QUẢ WATERFALL TRACE LOG (SAU KHI CHẠY TEST SUITE TRÊN API THẬT)

> ⚠️ **YÊU CẦU NGHIỆM THU:** Mở tệp `.env` điền `GEMINI_API_KEY` (hoặc `OPENAI_API_KEY`) để kết nối LLM thật trước khi thực thi `python src/app.py --all`. Bài nộp chỉ dùng Mock Offline Provider sẽ không đạt điểm nghiệm thực tế.

**Cấu hình chạy nghiệm thu:** `LLM_PROVIDER=gemini`, `LLM_MODEL=gemini-3.5-flash-lite` (Google Gemini API thật, Native Tool Calling). Lệnh: `python src/app.py --all` — 5/5 Test Cases pass trong **một lần chạy liên tục**.

Dán 1 đoạn trích xuất log tiêu biểu từ file `docs/trace_waterfall.json` sinh ra từ phản hồi LLM API thật — **TC04 (multi-step reasoning)**, Agent tự tra cứu cố vấn rồi dùng chính Observation đó để đặt lịch:

```json
[
  {
    "step": 1,
    "action_type": "TOOL_EXECUTION",
    "thought": "Gemini quyết định gọi công cụ 'academic_query' với tham số: {\"student_id\": \"SV2026001\"}",
    "tool_name": "academic_query",
    "arguments": { "student_id": "SV2026001" },
    "observation": {
      "status": "SUCCESS",
      "student_id": "SV2026001",
      "data": { "full_name": "Nguyễn Văn An", "class": "AI-K4", "gpa": 3.85, "status": "Đang học", "advisor": "PGS.TS Nguyễn Văn A" }
    },
    "latency_ms": 2436.62,
    "tool_latency_ms": 0.0
  },
  {
    "step": 2,
    "action_type": "TOOL_EXECUTION",
    "thought": "Gemini quyết định gọi công cụ 'schedule_appointment' với tham số: {\"student_id\": \"SV2026001\", \"datetime_str\": \"14:00 15/09/2026\", \"advisor_name\": \"PGS.TS Nguyễn Văn A\"}",
    "tool_name": "schedule_appointment",
    "arguments": { "student_id": "SV2026001", "datetime_str": "14:00 15/09/2026", "advisor_name": "PGS.TS Nguyễn Văn A" },
    "observation": {
      "status": "SUCCESS",
      "booking_id": "BK-SV2026001-99",
      "advisor": "PGS.TS Nguyễn Văn A",
      "datetime": "14:00 15/09/2026"
    },
    "latency_ms": 1668.1,
    "tool_latency_ms": 0.0
  },
  {
    "step": 3,
    "action_type": "FINAL_ANSWER",
    "thought": "Gemini phản hồi trực tiếp bằng văn bản (không cần gọi công cụ).",
    "output": "Dựa trên thông tin từ hệ thống học vụ VinUni, tôi đã hoàn thành yêu cầu của bạn: 1. Tra cứu thông tin cố vấn học tập: Sinh viên Nguyễn Văn An (Mã: SV2026001) — Cố vấn học tập: PGS.TS Nguyễn Văn A. 2. Đặt lịch hẹn tư vấn: 14:00 ngày 15/09/2026 — Mã đặt lịch: BK-SV2026001-99 — Trạng thái: Thành công. ...",
    "latency_ms": 5563.43
  }
]
```

**Bảng tổng hợp 5 Test Cases (API thật):**

| TC | Chuỗi ReAct thực tế | Kết quả |
| :--- | :--- | :---: |
| TC01 | FINAL_ANSWER trực tiếp (không gọi Tool) | ✅ Đúng kỳ vọng |
| TC02 | `academic_query(SV2026001)` → FINAL_ANSWER | ✅ |
| TC03 | `schedule_appointment(SV2026002, 09:00 20/09/2026, TS. Lê Thị B)` → FINAL_ANSWER (BK-SV2026002-99) | ✅ |
| TC04 | `academic_query(SV2026001)` → `schedule_appointment(..., advisor lấy từ Observation)` → FINAL_ANSWER | ✅ Multi-step |
| TC05 | `academic_query(SV9999999)` → NOT_FOUND → FINAL_ANSWER báo không tìm thấy, không bịa GPA | ✅ Edge case |

**Nhận xét quan sát (Observability):**
- Mỗi bước gọi LLM mất ~1.3–5.6s (bước FINAL_ANSWER tổng hợp dài nhất: TC04 5.56s, TC01 4.84s), trong khi `tool_latency_ms` của MCP Server ~0ms → nút cổ chai là LLM, không phải Tool. Tổng TC04 (3 bước) ≈ 9.7s.
- Trong các lần chạy thử trước đó gặp lỗi tạm thời từ Gemini (`429 RESOURCE_EXHAUSTED`, `503 UNAVAILABLE`, `Server disconnected`) khiến 1 Test Case bị ghi `LLM_API_ERROR`. Đã bổ sung retry có chờ cho các lỗi tạm thời này; lần chạy nghiệm thu cuối không cần retry và pass 5/5.
- TC01: câu hỏi chung, model trả lời trực tiếp không gọi Tool — đúng quyết định "Chatbot là đủ" trong sơ đồ Agentic Fit. Lưu ý: nội dung quy chế là kiến thức chung của LLM, không được xác thực bằng Tool (hạn chế của Cấp 2; có thể bổ sung Tool tra cứu văn bản quy chế nếu cần độ chính xác).

**Cải tiến so với starter code:**
1. ReAct Loop thật: Observation được nạp lại cho LLM qua scratchpad, vòng lặp chỉ dừng khi LLM trả FINAL_ANSWER (starter `break` ngay sau Tool Call đầu tiên nên không chạy được multi-step).
2. Bỏ cơ chế fallback im lặng về Mock khi đã có API Key thật — lỗi API được ghi vào trace (`LLM_API_ERROR`) thay vì trộn dữ liệu Mock vào kết quả nghiệm thu.
3. Retry có chờ cho lỗi tạm thời (429/503/mất kết nối); guard `MAX_ITERATIONS_REACHED` chống vòng lặp vô hạn.

---

## 3. TỔNG KẾT KẾT QUẢ NGHIỆM THU & NỘP BÀI

- [x] Đã điền API Key thật trong `.env` và xác nhận Agent chạy mượt mà trên LLM API thật (Gemini/OpenAI).
- **Tổng số Test Cases đã chạy thành công:** 5 / 5 test cases.
- **Số lượt gọi Tool qua MCP Server chính xác:** 5 lượt (TC02: 1, TC03: 1, TC04: 2, TC05: 1 — đúng tên Tool và tham số; TC01 đúng khi không gọi Tool).
- **Kết quả đẩy Repo nộp bài:** [x] Đã Commit và Push mã nguồn thành công lên GitHub cá nhân.

---

> ✅ **HOÀN TẤT NỘP BÀI:** Sao chép đường link GitHub Repository cá nhân của bạn và dán vào ô nộp bài trên hệ thống LMS VLearn để hoàn tất Bài Lab 3!
