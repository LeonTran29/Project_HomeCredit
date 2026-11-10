# Data Quality Log - Project Olist

Log of data quality issues found during profiling.
Finding template: Numbers -> Reading -> So what -> Limits.

---

## DQ-001: Cột cờ (flag) lẫn kiểu BOOL và INT64 trong application_train
- **Numbers**: 3 cột BOOL (FLAG_OWN_CAR, FLAG_OWN_REALTY, EMERGENCYSTATE_MODE); các cột cờ còn lại (FLAG_MOBIL, FLAG_EMP_PHONE, FLAG_WORK_PHONE, FLAG_CONT_MOBILE, FLAG_PHONE, FLAG_EMAIL, FLAG_DOCUMENT_2..21, REG_*/LIVE_*_NOT_*) là INT64 0/1. Tổng 307.511 dòng.
- **Reading**: Cùng bản chất Yes/No nhưng lưu hai kiểu khác nhau. Nguồn CSV là chữ (Y/N, Yes/No) hoặc 0/1; BigQuery Auto detect tự chọn kiểu theo từng cột. Chưa phải lỗi giá trị, mới chỉ là không nhất quán kiểu. Chưa kiểm tỷ lệ NULL của EMERGENCYSTATE_MODE.
- **So what / Action**: AVG(Flag) ra thẳng tỷ lệ với INT64 nhưng BOOL phải ép kiểu trước; kiểu lẫn lộn dễ gây lỗi khi viết SQL hàng loạt và khi đưa vào Power BI. Chuẩn hoá toàn bộ về INT64 0/1, giữ NULL là NULL.
- **Fix location**: stg (homecredit_stg.application_train), dùng CAST. Raw giữ nguyên.
- **Status**: OPEN

---

## DQ-002: DAYS_REGISTRATION là FLOAT64, có 1 dòng giá trị lẻ
- **Numbers**: 1/307.511 dòng lẻ: SK_ID_CURR = 408583, DAYS_REGISTRATION = -10116.041666 (phần lẻ ≈ 1/24, tức 1 giờ; suy luận của tôi, chưa có tài liệu xác nhận). Kiểm các cột số đếm khác (OBS_30, CNT_FAM_MEMBERS, AMT_REQ_CREDIT_BUREAU_YEAR): chưa có kết quả.
- **Reading**: Cột đo số ngày, lẽ ra là số nguyên. Một dòng có độ chính xác theo giờ trong khi các dòng còn lại tính theo ngày nguyên. Ảnh hưởng không đáng kể.
- **So what / Action**: CAST thẳng sang INT64 sẽ làm tròn dòng này mà không báo; ghi rõ quyết định. Đề xuất ROUND rồi ép INT64 (dòng 408583 thành -10116). Mức độ: thấp.
- **Fix location**: stg (homecredit_stg.application_train).
- **Status**: OPEN

---

## DQ-003: DAYS_EMPLOYED có giá trị dương bất thường (mã 365.243)
- **Numbers**: 55.374 dòng dương, tất cả bằng 365.243 (n_positive_other = 0); thuộc Pensioner 55.352 (vỡ nợ 5,39%) và Unemployed 22 (36,36%, mẫu nhỏ).
- **Reading**: bỏ chữ “chưa xác nhận bằng NAME_INCOME_TYPE”, thay bằng “xác nhận: mã chỉ xuất hiện ở hai nhóm không có việc làm”.
- **So what / Action**: Ở stg, thay 365.243 bằng NULL và thêm cờ is_employed_unknown. Giữ nguyên dòng. Báo tỷ lệ vỡ nợ của nhóm này riêng, kèm mẫu số 55.374, so với base rate 8,07%.
- **Fix location**: stg (homecredit_stg.application_train).
- **Status**: OPEN
