#Chủ đề: Biên dịch chéo thư viện, ứng dụng và tích hợp vào hệ thống Buildroot.
Thiết bị mục tiêu: BeagleBone Black (Kiến trúc ARM Cortex-A8)

Mục tiêu cốt lõi của bài tập này là hiểu và làm chủ được quy trình phát triển phần mềm cho hệ thống nhúng Linux thông qua công cụ Buildroot. Quy trình này bao gồm: sử dụng thư viện nguồn mở có sẵn, tự phát triển thư viện cá nhân, so sánh các cơ chế liên kết (Linking), và cuối cùng là đóng gói ứng dụng thành một module hệ thống.

PHẦN 1: BIÊN DỊCH ỨNG DỤNG VỚI THƯ VIỆN CÓ SẴN (cJSON)

Mục đích: Khởi tạo cấu hình Buildroot để tự động tải thư viện cJSON, sau đó sử dụng Trình biên dịch chéo (Cross-compiler) để dịch mã nguồn C ra file thực thi cho chip ARM.

1. Mã nguồn chương trình HelloJSON

Chương trình sử dụng thư viện cJSON để khởi tạo và in ra một chuỗi JSON.
Lệnh thực hiện mở code: nano my_projects/src/HelloJSON.c
<img width="786" height="533" alt="image" src="https://github.com/user-attachments/assets/8d09e8b5-6533-465e-90b6-1c02f49c9c2c" />

2. Biên dịch chéo và Kết quả thực thi

Lệnh biên dịch chéo sử dụng Toolchain của Buildroot:

arm-buildroot-linux-gnueabihf-gcc HelloJSON.c -I../../output/staging/usr/include -L../../output/staging/usr/lib -lcjson -o HelloJSON


(Giải thích: Cờ -I trỏ đến thư mục chứa file Header, cờ -L trỏ đến thư mục chứa file thư viện tĩnh/động, cờ -lcjson yêu cầu Linker liên kết thư viện vào chương trình).

Kết quả khởi chạy thành công trên bo mạch BeagleBone Black:

PHẦN 2: TỰ TẠO THƯ VIỆN CÁ NHÂN VÀ SO SÁNH (STATIC vs DYNAMIC)

Mục đích: Tự viết thư viện toán học (mylib), biên dịch thành 2 chuẩn tĩnh (.a) và động (.so). Sau đó phân tích chuyên sâu sự khác biệt về dung lượng và yêu cầu phụ thuộc của hệ điều hành.

1. Mã nguồn thư viện và chương trình Test

Chương trình gọi hàm add() từ thư viện cá nhân để tính tổng.
Lệnh thực hiện mở code: nano my_projects/src/main_test.c
<img width="786" height="533" alt="image" src="https://github.com/user-attachments/assets/04d776d2-943f-40b3-b602-c7d47234aea1" />

<img width="786" height="533" alt="image" src="https://github.com/user-attachments/assets/be7a755b-c1fa-467f-a152-d46611632531" />

<img width="786" height="533" alt="image" src="https://github.com/user-attachments/assets/f0749570-8bc7-43c7-aba1-21ba82ea729c" />
2. Đánh giá sự khác biệt về Kích thước (Dung lượng file)

Lệnh thực thi trên Board: ls -lh /usr/bin/test_static /usr/bin/test_dynamic

Giải thích: Trình liên kết (Linker) đã trích xuất toàn bộ mã máy của hàm từ libmylib.a và nhúng (embed) vào bên trong file test_static, khiến file này phình to gấp nhiều lần. Ngược lại, bản test_dynamic rất nhẹ vì chỉ chứa bảng tham chiếu đến file .so bên ngoài.
<img width="821" height="521" alt="image" src="https://github.com/user-attachments/assets/23584269-4625-4f13-89ea-2ab199f32b20" />

3. Đánh giá sự khác biệt về Phụ thuộc (Dependencies)

Do Buildroot tối ưu hóa loại bỏ công cụ debug trên Board, ta sử dụng Cross-readelf trên máy ảo Ubuntu để phân tích Header của tệp ELF.
Lệnh thực thi trên Ubuntu: ./output/host/bin/arm-buildroot-linux-gnueabihf-readelf -d output/target/usr/bin/test_dynamic | grep NEEDED
<img width="786" height="533" alt="image" src="https://github.com/user-attachments/assets/2f720990-4401-4023-b4a6-c1654522c906" />
Giải thích: File test_static hoạt động hoàn toàn độc lập. Trong khi đó, file test_dynamic bắt buộc hệ điều hành phải cung cấp file libmylib.so và libc.so.6 khi chạy.

PHẦN 3: TÍCH HỢP ỨNG DỤNG VÀ THƯ VIỆN VÀO BUILDROOT

Mục đích: Thay vì copy file thủ công, ta biến ứng dụng thành một Package chính quy của Buildroot. Hệ thống sẽ tự động giải quyết các ràng buộc phụ thuộc (Dependency), tự động biên dịch và nén vào hệ điều hành.

1. Mã nguồn ứng dụng FinalApp

Chương trình kết hợp cả thư viện mylib (để tính toán) và cJSON (để đóng gói kết quả).
<img width="786" height="533" alt="image" src="https://github.com/user-attachments/assets/dd891358-045d-4797-b279-553bc3ee16c5" />

2. Cấu hình Package cho Buildroot

Tạo file cấu hình Kconfig (Config.in) và Makefile (finalapp.mk).
Lệnh thực hiện: Sử dụng lệnh select BR2_PACKAGE_CJSON để ép Buildroot phải ưu tiên biên dịch thư viện cJSON trước khi biên dịch ứng dụng này.
<img width="786" height="533" alt="image" src="https://github.com/user-attachments/assets/6a16741a-d28f-45dd-aa81-fc18b93e0e13" />

<img width="786" height="533" alt="image" src="https://github.com/user-attachments/assets/3590f4d2-67eb-4bc0-9968-8afacfbaff5a" />

3. Kết quả chạy FinalApp trên bo mạch

Sau khi chạy lệnh make để hệ thống tự động Build và chép ra thẻ nhớ sdcard.img, ứng dụng đã tích hợp thành công vào nhân hệ điều hành.
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/7ad80071-2c7a-4ee9-8fb8-a48bb13ec435" />

PHẦN 4: MINH CHỨNG KIẾN TRÚC THƯ MỤC BUILDROOT

Để minh chứng cho việc hiểu sâu về nguyên lý tổ chức file của Buildroot, dưới đây là hình ảnh kiểm tra các thư mục cốt lõi trên máy ảo Ubuntu.

Lệnh thực hiện: ls -l output/target/usr/bin và ls -l output/target/usr/lib

<img width="921" height="515" alt="image" src="https://github.com/user-attachments/assets/01fac5ac-20d6-46e7-8bab-3e7da6d5b547" />
<img width="921" height="141" alt="image" src="https://github.com/user-attachments/assets/8ad9d200-98e9-419f-ac70-a0849de3e6f7" />

<img width="921" height="515" alt="image" src="https://github.com/user-attachments/assets/230c1198-46ce-476f-9f01-0aca7383887f" />
Giải thích: > * Thư mục Staging đóng vai trò là Sysroot chứa các Header và Library phục vụ quá trình Compile.

Thư mục Target đóng vai trò là bản nháp của hệ thống RootFS. Mọi ứng dụng sau khi biên dịch xong đều được chèn vào đây trước khi nén thành file sdcard.img.
Video Demo:
https://drive.google.com/file/d/1GRx4i2sXR0ASKQf_AOxu0R45blrvTqTB/view?usp=sharing
