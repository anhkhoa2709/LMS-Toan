# Bàn giao LMS "Lớp Toán thầy Khoa" (16/10/2026)

Tài liệu cho phiên làm việc kế tiếp. Xưng "tôi", gọi "thầy", viết tiếng Việt.

## 0. Quy tắc bắt buộc (giữ nguyên từ các phiên trước)

- Không tự chạy DELETE / DROP / ALTER trên cơ sở dữ liệu; phải hỏi thầy trước.
- Thay trang bài học thì sao lưu trước vào bảng `backup_lp_<x>_<ngày>`.
- Không gõ mật khẩu; thầy tự đăng nhập. KHÔNG xóa thư mục `.vercel` trên máy thầy.
- Biến môi trường AI chỉ nêu tên, không ghi giá trị.
- Cơ sở dữ liệu: Neon project `young-river-09203581`, chỉ thao tác qua Neon MCP.
- Soạn bài: không sửa đề; không thêm gợi ý nếu không được yêu cầu; không dùng chữ "lấy"; mỗi phép biến đổi một dòng; câu dẫn trước danh sách kết thúc bằng ":"; kí hiệu mới phải khai báo; không dùng `\quad`; không ghi "do Claude soạn" trong bài.
- Trình chiếu: đề 2 phút → trang Ý tưởng (sơ đồ, hình hiện dần, ít chữ) → trang Lời giải chép lại đề. Không trang nào được tràn khung: lời giải dài thì tách sang trang "(tiếp)", khoảng ≤ 400 ký tự hiển thị mỗi trang khi có hình bên trái.
- Bài tập về nhà: chỉ gắn qua `lessons.bai_tap_id` + `cham_thang` (mỗi bài 2 điểm, tổng 10), KHÔNG thêm trang bài tập vào bài học.

## 1. Trở ngại lớn nhất: chưa có mã nguồn LMS

- Kho GitHub duy nhất là `anhkhoa2709/LMS-Toan` (công khai). Kho này KHÔNG chứa mã LMS:
  - `master`: game thử "Tây Du" (`tay-du-ky/`) và 50 ảnh chân dung thầy tải lên (`background/1.png … 50.png`).
  - `claude/lms-lop-toan-handover-8wv8wn`: phù hiệu (`phu-hieu/`), 50 avatar đã nén (`avatars/01.webp … 50.webp`), tệp bàn giao này.
- Vercel MCP thấy dự án `mathflow-lms-vercel` (team `khoa2709`, id `prj_QhNrHsfrV0kLXxD5y7BXHEfAnmRK`) nhưng mọi lệnh đọc deployment đều 403 (thiếu quyền scope `khoa2709`).
- Đã hướng dẫn thầy 3 cách: (1) đẩy thư mục mã LMS trên máy lên một kho GitHub riêng tư (nhớ `.gitignore` có `.env*`, `.vercel`, `node_modules`); (2) chạy `claude remote-control` trong thư mục dự án; (3) kết nối lại Vercel ở https://claude.ai/customize/connectors và cấp quyền team `khoa2709`, rồi mở phiên mới.
- Khi có mã nguồn, các việc giao diện ở mục 3 làm được ngay.

## 2. Đã làm trong phiên này

### Nội dung bài học (9CT1)
- `tu-giac-noi-tiep-9ct1` (ẩn): 94 trang, hình SVG nhúng thẳng (data URI), lời giải đã tách trang "(tiếp)". Bài tập id 206 (15 bài). Sao lưu trước khi tách trang: `backup_lp_tgnt9ct1_20261009`. Đã sửa: mục 5 định lí thành HA·HC = HB·HD; Bài 5 b) đề có thể nhầm (ghi chú trong đáp án).
- `dinh-li-viete-9` (ẩn): 18 trang lý thuyết/ý tưởng đổi sang sơ đồ; đã bỏ 11 trang bài tập trong bài; bài tập id 207 (10 bài). Sao lưu: `backup_lp_viete_20261009`.
- Bài cũ `tu-giac-noi-tiep` (bài tập 51) GIỮ NGUYÊN theo ý thầy. Bảng tạm `hinh_tam` đã xóa (thầy cho phép).

### Kéo co
- Đặc tả thăng hạng Bàn tay Đồng/Bạc/Vàng: https://claude.ai/artifact/C24fGhLdMG8qbsHvnmmkoz
- Bảng `keo_co_hang`, `keo_co_hang_log` đã có trong cơ sở dữ liệu (do người làm LMS tạo sau đó). Trận đấu với máy lưu ở `tro_choi_luot`, không tính hạng.

### Kế hoạch khối (phiên 2, mục 1)
- Bảng `ke_hoach` (tên, khối, `lop[]`, `mach`, `bat_dau`, `ket_thuc`) và `buoi_hoc` (`ke_hoach_id`, `stt`, `tieu_de`, `lesson_slug`, `assignment_id`).
- View `lich_buoi`: tự tính ngày từng buổi từ `lich_lop` (thu: 1 = thứ Hai … 7 = Chủ nhật, theo đúng `mach`), trừ `ngay_nghi`, cộng `buoi_bu`. Trạng thái bài "đã mở" dựa trên `lessons.published`, `lop`, và `han_lop` (loai `bh`/`bt`).
- 4 kế hoạch đã nhập từ bảng cũ `buoi_bai` (bảng này giữ nguyên): 10CT1 Đại số chuyên (56/57 buổi), 10CT1 Tổ hợp chuyên (49/58), 12K1 HSG Thành phố (16/34), 8CT1 (23/66). 20 buổi đã gắn bài học theo tên trùng khớp.

### Hôm nay / Tiến độ (phiên 2, mục 2)
- View `hom_nay`: các buổi hôm nay (giờ Việt Nam), trạng thái bài và bài tập.
- View `tien_do`: buổi theo lịch, buổi đã mở bài, chênh lệch (âm = chậm), việc cần làm 2 tuần tới.
- Giao diện trong LMS: CHƯA làm (thiếu mã nguồn).

### Avatar học sinh
- Bảng `avatar` (id 1–50, url trỏ tới `avatars/NN.webp` trên nhánh `claude/lms-lop-toan-handover-8wv8wn`, cột `hien`).
- Bảng `hoc_sinh_avatar` (user_id, avatar_id, cap_nhat). Không thêm cột vào `users`.
- Lưu ý: nếu xóa/đổi tên nhánh thì đường dẫn ảnh hỏng; tốt nhất chép `avatars/` vào mã LMS rồi cập nhật cột `url`. Nên kiểm tra giấy phép bộ ảnh.
- Giao diện chọn avatar: CHƯA làm (thiếu mã nguồn).

### Quà chuỗi
- Bảng `qua_chuoi` (lop, moc = ô thứ mấy, ten, bieu_tuong, hien, created_at, updated_at; unique(lop, moc)). Chưa có dòng nào.
- Bản chạy thử giao diện (giáo viên chỉnh quà theo lớp; học sinh thấy thanh chuỗi như trục số, quà mới rơi xuống treo vào ô): https://claude.ai/artifact/GQnwXfsxLNQQxSHgHMukhP
- Chuỗi hiện tính theo số câu/ngày ở `site_content.streak_so_cau` (đang là 8).
- Hiệu ứng "quà mới": so `qua_chuoi.updated_at` với lần cuối học sinh mở thanh chuỗi.

### Khác
- Phù hiệu "LỚP TOÁN THẦY KHOA" (khung trắng, xung quanh trong suốt): `phu-hieu/phu-hieu.png`, `phu-hieu/phu-hieu.svg`. Gửi cho học sinh dạng tệp hoặc link raw GitHub để không mất nền trong suốt.

## 3. Việc còn lại

Thầy cần cung cấp:
1. Mã nguồn LMS (mục 1).
2. Phân phối chương trình: 9CT1, 11K1, 12K1 buổi thường (thứ 2, thứ 4), 8CT1 các buổi thứ Năm.
3. Lịch lớp 10CA-TH1 và 10CL1 (chưa có trong `lich_lop`).
4. Chốt: "nhanh/chậm" tính theo bài đã mở (hiện tại) hay thêm bảng điểm danh "đã dạy buổi".
5. Rà 4 kế hoạch đã nhập và chỉ bài nào ứng với buổi nào (các buổi chưa gắn bài đang hiện "chưa soạn").
6. Duyệt bản chạy thử quà chuỗi; cho biết quà thật của từng lớp.

Khi có mã nguồn, làm trong LMS:
- Trang Hôm nay và Tiến độ (đọc `hom_nay`, `tien_do`).
- Màn hình chọn avatar và hiện avatar cạnh tên.
- Nút chỉnh quà chuỗi theo lớp và thanh chuỗi học sinh (theo bản chạy thử).
- Phần hạng kéo co nếu giao diện chưa có (theo đặc tả).
- Lỗi "Application error: a client-side exception" học sinh gặp nhiều lần: gợi ý bật Skew Protection trên Vercel và thêm `app/global-error.tsx`; cần mã nguồn để xử lý tận gốc.
- Ghi nội dung tệp này vào `BAN-GIAO-AI.md` trong mã LMS.
