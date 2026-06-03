# Tài Liệu Phạm Vi & Kế Hoạch Triển Khai: Voice-to-Cart (Grab)

## 1. Track, Product/App và Đối tượng Người dùng
* **Track:** Multimodal (Track D - Tự chọn)
* **Product/App thực tế:** Grab (Tích hợp tính năng Voice-to-Cart)
* **User cụ thể:** Tài xế Grab đang di chuyển trên đường.
* **Xác thực người dùng:** **Có**. Nhóm đã thực hiện phỏng vấn nhanh các tài xế thực tế để thu thập dữ liệu gốc.

---

## 2. Tóm tắt Bằng chứng (Evidence Summary)

| Bằng chứng (Evidence) | Nguồn | User/Pain nói lên điều gì? | SPEC phải thay đổi gì? |
| :--- | :--- | :--- | :--- |
| Thao tác tay khi xe rung lắc dễ bấm nhầm. | Tự trải nghiệm | Ma sát vật lý cực lớn, gây nguy hiểm trực tiếp khi lái xe. | Chuyển từ **Automation** sang **Augmentation** để giảm thiểu rủi ro. |
| Sợ AI đặt nhầm 10 suất thay vì 1 suất. | Phỏng vấn tài xế | Nỗi sợ về rủi ro tài chính do AI nghe nhầm trong môi trường ồn ào. | Bổ sung thêm luồng **Voice-Confirm** (Xác nhận bằng giọng nói). |

---

## 3. Tuyên bố về Nỗi đau (Pain Statement)
> ⚠️ **Core Pain Statement:**
> Đối tượng tài xế Grab đang gặp khó khăn lớn ở bước **soạn giỏ hàng** và **kiểm tra món ăn**. Việc thao tác bằng tay khi đang điều khiển phương tiện rất nguy hiểm, đồng thời tiếng ồn từ môi trường dễ gây nhầm lẫn thông tin, dẫn tới rủi ro tai nạn hoặc đặt sai đơn hàng gây lãng phí. 
> 
> *Bằng chứng cốt lõi từ phỏng vấn trực tiếp:* `"Dừng xe đặt đồ ăn mất 5-10 phút thu nhập, rất bất tiện"`.

---

## 4. Phân đoạn Xây dựng (Build Slice)
* **Giải pháp kiểm chứng:** Dành cho tài xế Grab đang cần đặt món ăn rảnh tay. 
* **Phạm vi Prototype:** Sử dụng AI để **Augment (Tăng cường)** hành động soạn giỏ hàng và đọc lại nội dung bằng âm thanh để xác nhận (phản hồi âm thanh), đồng thời xử lý lỗi nghe nhầm số lượng do tiếng ồn bằng cơ chế xác nhận lại (**Voice-Confirm**).

---

## 5. Quyết định Tự động hóa / Tăng cường (Auto/Aug Decision)
* **Lựa chọn:** **Augmentation** *(AI gợi ý / Draft / Phân loại; User đưa ra quyết định cuối cùng)*.
* **Lý do kỹ thuật:** Độ chính xác của công nghệ *Voice-to-Text* trong môi trường tiếng ồn đô thị không đạt ngưỡng tin cậy tuyệt đối (>95%) để vận hành tự động hóa hoàn toàn.
* **Vai trò của con người (Human Role):** **Decider** *(Người dùng nói "Đúng" hoặc "Sai" trước khi món ăn chính thức được đưa vào giỏ)*.

---

## 6. Kịch bản 4 Nhánh (Four Paths)

| Luồng đi (Path) | Prototype phải thể hiện được điều gì? |
| :--- | :--- |
| 🟢 **Happy Path** | User ra lệnh gọi món $\rightarrow$ AI nhận diện và thêm đúng $\rightarrow$ User xác nhận bằng giọng nói $\rightarrow$ Thành công. |
| 🟡 **Low-confidence Path** | AI nghe không rõ số lượng $\rightarrow$ AI chủ động hỏi lại: *"Bạn muốn đặt 1 hay 2 phần?"*. |
| 🔴 **Failure Path** | Tiếng còi xe/tạp âm quá lớn $\rightarrow$ AI nhận diện không thể xử lý $\rightarrow$ Thông báo lỗi bằng âm thanh và yêu cầu user thao tác lại khi đã dừng xe an toàn. |
| 🔵 **Correction Path** | AI đặt nhầm món $\rightarrow$ User phản hồi: *"Không, bún chả cơ"* $\rightarrow$ AI lập tức nhận diện lỗi và sửa lại giỏ hàng. |

---

## 7. Kịch bản Lỗi Nguy Hiểm Nhất (Critical Failure Mode)
* **Tình huống kích hoạt (Trigger):** User đang lái xe qua khu vực có tiếng còi xe hoặc tạp âm lớn.
* **Lỗi hệ thống (Failure):** AI trích xuất sai số lượng hoặc nhầm tên món ăn.
* **Hậu quả (Impact):** Đặt sai đơn hàng gây mất tiền vô lý và làm mất niềm tin nghiêm trọng từ người dùng.
* **Giải pháp xử lý trên Prototype:** Cơ chế **Ask Again** *(Yêu cầu xác nhận lại toàn bộ danh sách món bằng âm thanh trước khi chốt đơn)*.
* **Chủ biên kiểm thử (Owner):** `[Tên thành viên phụ trách Testing]`

---

## 8. Kế hoạch Phân công Nhiệm vụ (Owner Plan cho Sáng Day 06)

| Thành viên | Trách nhiệm chính | Bằng chứng bắt buộc phải có trong Repo |
| :---: | :--- | :--- |
| **Thành viên 1** | Research / Evidence | File `Evidence Pack (.md)` kèm các tệp ghi âm/video phỏng vấn thực tế. |
| **Thành viên 2** | SPEC | File `AI Product Canvas` và tài liệu chi tiết `User Stories` cho 4 paths. |
| **Thành viên 3** | Prototype | Bản `Agent Script` chạy trên LangGraph hoặc giao diện tương tác Mockup. |
| **Thành viên 4** | Test / Failure path | Bảng `Eval Metrics` cùng kết quả log chạy kiểm thử cho cả 4 paths. |
| **Thành viên 5** | Demo script / Repo | Video quay Demo sản phẩm hoàn chỉnh và tài liệu hướng dẫn triển khai (`README.md`) trên GitHub. |