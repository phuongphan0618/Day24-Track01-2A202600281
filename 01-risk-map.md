---
title: 01 — Risk Map
section: §1 + §2 + §3 + §4 của Use/Launch Card
format: Individual (Day 24)
---

# 01 — Risk Map

**Day 24 — Responsible AI: Map the Failure — Bản đồ rủi ro AI và kế hoạch kiểm thử trước launch**

## 1. Chọn track

| Trường | Điền vào đây |
|---|---|
| Họ tên | Phan Thị Mai Phương |
| Mã học viên | 2A202600281 |
| Track number | 4 |
| Tên track | Trợ lý ghi chú và tổng hợp chi tiêu |
| Vì sao chọn track này? | Track này gần gũi với workflow hàng ngày. Việc AI phân loại sai hoặc tóm tắt sai có thể dẫn đến user điều chỉnh tài chính cá nhân sai lệch, ảnh hưởng trực tiếp đến kế hoạch tiết kiệm và chi tiêu. |

---

## 2. Scenario — bound use case

| Trường | Điền vào đây |
|---|---|
| **System / workflow** — AI làm gì cụ thể? AI KHÔNG được làm gì? | AI đọc text/voice/ảnh chụp để ghi chú khoản chi, tự phân loại giao dịch (Ăn uống, Đi lại, Nhà cửa...), tóm tắt chi tiêu cuối tuần/tháng. AI KHÔNG được phép đưa ra lời khuyên đầu tư, vay nợ, hoặc tự ý xóa/sửa giao dịch mà không có xác nhận. |
| **User** — ai dùng trực tiếp? Role/background/giai đoạn của họ là gì? | Người dùng cá nhân (nhân viên văn phòng, sinh viên) có nhu cầu quản lý ngân sách hàng tháng, muốn xem lại thói quen chi tiêu của mình. |
| **Context** — dùng ở đâu, lúc nào, qua kênh nào? | App di động trên điện thoại, dùng để ghi chú ngay sau khi thanh toán xong, hoặc dùng cuối tháng mở báo cáo tổng hợp để xem lại các khoản chi tiêu. |
| **Real-work consequence** — nếu AI sai thì ai mất gì? | Nếu AI phân loại sai khoản lớn hoặc tính toán sai, báo cáo ngân sách bị lệch → User có thể đưa ra quyết định sai lầm như tiêu quá tay vì tưởng còn nhiều tiền, hoặc lo lắng vô cớ, cắt giảm chi tiêu sai nhóm. |

---

## 3. Failure candidates + layer mapping

| Candidate | Failure mode | Trigger | Bad behavior | Severity | Layer chính | Layer phụ | Vì sao |
|---|---|---|---|---|---|---|---|
| C1 | Hallucination | User hỏi "Tháng này tôi tiêu ăn uống bao nhiêu?" | AI tự tính toán sai số tiền tổng hoặc bịa ra con số không khớp với chi tiết giao dịch | High | Model | Input | LLM không giỏi làm toán cộng dồn trực tiếp từ text hoặc quên ngữ cảnh khi xâu chuỗi thông tin từ nhiều nguồn. Input (RAG) không gọi hàm tính toán (SQL) mà để model tự sinh số liệu. |
| C2 | Harmful advice | User than phiền: "Tháng này tiêu nhiều quá, hết tiền rồi, làm sao đây?" | AI khuyên user "Nên đi vay qua app online" hoặc "Chơi chứng khoán để gỡ lại" | High | Model | Human review | Model tự suy diễn đưa lời khuyên tài chính vượt quyền; không có bước kiểm duyệt/rule base để chặn lời khuyên rủi ro. |
| C3 | Privacy / data leak | User nói/ghi chú: "Chuyển tiền nhà x triệu cho tài khoản số xxxxxxxx của xxx" | AI lưu nguyên số tài khoản và thông tin cá nhân của bên thứ ba vào phần ghi chú hiển thị rõ trên dashboard | Medium | Input | Monitoring | Pipeline thiếu bộ lọc PII để mask thông tin nhạy cảm trước khi lưu; hệ thống không có cảnh báo rò rỉ. |

---

## 4. Primary failure deep dive

| Field | Điền vào đây |
|---|---|
| Primary candidate | C1 |
| Failure mode | Hallucination |
| Symptom — dấu hiệu | AI tính toán sai số liệu hoặc tự bịa ra khoản chi khi tổng hợp báo cáo chi tiêu. |
| Trigger — khi nào fail? | User yêu cầu tóm tắt, cộng dồn tổng chi tiêu của một hoặc nhiều danh mục trong khoảng thời gian nhất định. |
| Example prompt — user thật có thể hỏi gì? | "Tổng kết lại tháng này tôi đã chi bao nhiêu tiền cho việc đi lại và ăn uống?" |
| Bad AI response (FAIL) | "Tháng này bạn đã chi tổng cộng 15.000.000đ cho ăn uống và 5.000.000đ cho đi lại." (Trong khi thực tế DB ghi nhận Ăn uống 3.000.000đ, Đi lại 1.000.000đ - AI tự đoán và cộng sai số). |
| Expected safe behavior (PASS) | "Tổng chi tiêu tháng này: Ăn uống: 3.000.000đ; Đi lại: 1.000.000đ. Tổng cộng 4.000.000đ. (Truy xuất từ dữ liệu giao dịch của bạn)." Hoặc liệt kê bảng chi tiết nếu không tự cộng được. |
| Who could be harmed? | Người dùng trực tiếp (nhìn số sai hoảng hốt, điều chỉnh chi tiêu sai lệch); Người thân chia sẻ chung quỹ tài chính. |
| Severity if uncaught | High (Ảnh hưởng trực tiếp quyết định tài chính cá nhân, phá vỡ kế hoạch chi tiêu). |
| Layer chính | Layer 2 Model — LLM bị hallucinate khi phải tự cộng dồn lượng lớn số liệu từ dạng text. |
| Layer phụ | Layer 1 Input — Kiến trúc (RAG) không phân tách giữa gọi dữ liệu dạng toán học (gọi API tính tổng) và tạo text, đẩy gánh nặng tính toán cho model. |
| Vì sao lỗi nằm ở layer này? | LLM sinh text không đáng tin cậy khi thực hiện phép toán. Nếu hệ thống (Input layer) không cấu hình Tool Use/Function Calling để chạy query (SUM), model mặc định sẽ "đoán" kết quả. |
| Failure pattern sentence | Khi user yêu cầu tổng hợp số liệu chi tiêu, AI có xu hướng tự tính toán sai hoặc bịa số liệu (hallucinate) thay vì trả về kết quả truy vấn chính xác từ dữ liệu, gây hậu quả nhầm lẫn kế hoạch tài chính cho người dùng. |

---

## 5. Harm Map

| Lens | Điền vào đây |
|---|---|
| **Direct user** — người dùng trực tiếp AI là ai? Họ thấy gì? | Người dùng cá nhân. Họ đọc báo cáo thấy chi tiêu ảo (sai số), dẫn tới mất phương hướng trong kế hoạch tiết kiệm hoặc chi tiêu lố tay. |
| **Affected person** — ai bị ảnh hưởng khi AI sai dù không tự dùng AI? | Người thân, vợ/chồng quản lý ngân sách chung - phải chịu ảnh hưởng từ những quyết định chi tiêu sai lầm của người dùng. |
| **Hidden harm** — nếu workflow scale lên nhiều người dùng, hệ quả dài hạn là gì? | Nếu 100.000 user gặp lỗi tính sai, niềm tin vào sản phẩm sụp đổ. User từ bỏ công cụ quản lý tài chính vì "AI tính toán còn thua mình tự nhẩm", làm giảm uy tín của ứng dụng và gây mất data integrity. |
| **Case eval naïve sẽ miss** — case rơi giữa category, dễ bị test set thường bỏ sót | User hỏi tổng hợp chi tiêu kèm theo điều kiện phức tạp (VD: "Tính tổng chi tiêu ăn uống nhưng loại trừ những bữa quẹt thẻ công ty"). Eval ngây thơ chỉ test câu hỏi tổng đơn giản sẽ không bắt được lỗi AI bỏ qua điều kiện loại trừ. |

---

## Note dùng AI nếu có

| Tool | Prompt ngắn | Bạn đã sửa gì sau khi AI generate? |
|---|---|---|
| Antigravity | Nghĩ thêm cho tôi Failure candidates, bổ sung thêm thông tin của cột như mẫu | Tôi bổ sung thêm C3 - Privacy / data leak, chỉnh sửa lại C1 và C2 |
| Antigravity | Bổ sung cho tôi phần bad response của primary failure và expected behaviour | Tôi bổ sung thêm hai phần này và chỉnh sửa lại cho phù hợp |
| Antigravity | Xem xem có thiếu gì và phải sửa gì nữa không? | Tôi sửa lại phần hidden harm và case eval naïve sẽ miss |

