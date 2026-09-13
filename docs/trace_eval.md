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

**Cấu hình chạy nghiệm thu:** `LLM_PROVIDER=gemini`, `LLM_MODEL=gemini-2.5-flash-lite` (Google Gemini API thật, Native Tool Calling). Lệnh: `python src/app.py --all`.

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
    "latency_ms": 33518.96,
    "tool_latency_ms": 0.0
  },
  {
    "step": 2,
    "action_type": "TOOL_EXECUTION",
    "thought": "Gemini quyết định gọi công cụ 'schedule_appointment' với tham số: {\"advisor_name\": \"PGS.TS Nguyễn Văn A\", \"student_id\": \"SV2026001\", \"datetime_str\": \"14:00 15/09/2026\"}",
    "tool_name": "schedule_appointment",
    "arguments": { "advisor_name": "PGS.TS Nguyễn Văn A", "student_id": "SV2026001", "datetime_str": "14:00 15/09/2026" },
    "observation": {
      "status": "SUCCESS",
      "booking_id": "BK-SV2026001-99",
      "advisor": "PGS.TS Nguyễn Văn A",
      "datetime": "14:00 15/09/2026"
    },
    "latency_ms": 2350.13,
    "tool_latency_ms": 1.0
  },
  {
    "step": 3,
    "action_type": "FINAL_ANSWER",
    "thought": "Gemini phản hồi trực tiếp bằng văn bản (không cần gọi công cụ).",
    "output": "Thông tin của bạn đã được ghi nhận. Lịch hẹn tư vấn với Cố vấn học tập PGS.TS Nguyễn Văn A của bạn đã được đặt thành công vào lúc 14:00 ngày 15/09/2026. Mã đặt lịch là BK-SV2026001-99.",
    "latency_ms": 2129.19
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
- `latency_ms` cao bất thường (~33s, ~93s) ở một số bước là do thời gian chờ retry khi Gemini free tier trả về 429 (giới hạn request/phút); các bước bình thường ~1.7–2.5s. `tool_latency_ms` của MCP Server ~0–1ms → nút cổ chai là LLM, không phải Tool.
- Trong lần chạy `--all`, bước tổng hợp cuối của TC04 gặp lỗi `503 UNAVAILABLE` (model quá tải tạm thời). Đã bổ sung retry cho 503 và chạy lại riêng TC04 trên API thật; trace TC04 trong file là kết quả của lần chạy lại này.
- TC01: model không có dữ liệu quy chế VinUni nên trả lời hướng sinh viên liên hệ phòng Đào tạo thay vì bịa số liệu — phù hợp nguyên tắc Anti-Hallucination.

**Cải tiến so với starter code:**
1. ReAct Loop thật: Observation được nạp lại cho LLM qua scratchpad, vòng lặp chỉ dừng khi LLM trả FINAL_ANSWER (starter `break` ngay sau Tool Call đầu tiên nên không chạy được multi-step).
2. Bỏ cơ chế fallback im lặng về Mock khi đã có API Key thật — lỗi API được ghi vào trace (`LLM_API_ERROR`) thay vì trộn dữ liệu Mock vào kết quả nghiệm thu.
3. Retry có chờ cho lỗi 429/503; guard `MAX_ITERATIONS_REACHED` chống vòng lặp vô hạn.

---

## 3. TỔNG KẾT KẾT QUẢ NGHIỆM THU & NỘP BÀI

- [x] Đã điền API Key thật trong `.env` và xác nhận Agent chạy mượt mà trên LLM API thật (Gemini/OpenAI).
- **Tổng số Test Cases đã chạy thành công:** 5 / 5 test cases.
- **Số lượt gọi Tool qua MCP Server chính xác:** 5 lượt (TC02: 1, TC03: 1, TC04: 2, TC05: 1 — đúng tên Tool và tham số; TC01 đúng khi không gọi Tool).
- **Kết quả đẩy Repo nộp bài:** [x] Đã Commit và Push mã nguồn thành công lên GitHub cá nhân.

---

> ✅ **HOÀN TẤT NỘP BÀI:** Sao chép đường link GitHub Repository cá nhân của bạn và dán vào ô nộp bài trên hệ thống LMS VLearn để hoàn tất Bài Lab 3!
