# Báo cáo ngắn — Lab 16: CPU Instance thay thế GPU

## Lý do sử dụng CPU thay GPU
Tài khoản AWS mới tạo có quota GPU mặc định là 0 vCPU cho dòng G/VT instances. Yêu cầu tăng quota (Service Quotas → Running On-Demand G and VT instances) đã được gửi nhưng chưa được duyệt trong thời gian làm lab, nên chuyển sang phương án CPU với instance `t2.micro` và bài toán LightGBM thay vì LLM inference trên GPU.

## Kết quả benchmark trên CPU instance
- **Training time**: ~3.5 giây cho 284,807 rows — rất nhanh do LightGBM tối ưu tốt cho CPU, tận dụng histogram-based splitting.
- **AUC-ROC**: 0.9517 — mức cao, model phân biệt tốt giữa giao dịch gian lận và bình thường.
- **Inference latency**: ~0.43ms/row, throughput 1000 rows chỉ ~0.76ms — phù hợp real-time serving.

## So sánh CPU vs GPU
- **CPU phù hợp hơn** cho bài toán tabular/gradient boosting (LightGBM, XGBoost) vì các thuật toán này được tối ưu cho CPU, GPU không mang lại lợi thế rõ rệt.
- **GPU cần thiết** cho deep learning và LLM inference (vLLM + Gemma) vì các phép tính ma trận song song quy mô lớn chỉ GPU mới xử lý hiệu quả.
- Chi phí tương đương: `t2.micro` (free tier) vs `g4dn.xlarge` (~$0.526/giờ), nhưng GPU yêu cầu quota đặc biệt mà tài khoản mới khó được duyệt ngay.
