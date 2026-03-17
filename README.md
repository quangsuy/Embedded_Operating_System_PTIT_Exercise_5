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

<img width="786" height="533" alt="image" src="https://github.com/user-attachments/assets/dd891358-045d-4797-b279-553bc3ee16c5" />

<img width="786" height="533" alt="image" src="https://github.com/user-attachments/assets/6a16741a-d28f-45dd-aa81-fc18b93e0e13" />

<img width="786" height="533" alt="image" src="https://github.com/user-attachments/assets/3590f4d2-67eb-4bc0-9968-8afacfbaff5a" />

<img width="786" height="533" alt="image" src="https://github.com/user-attachments/assets/2f720990-4401-4023-b4a6-c1654522c906" />

<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/7ad80071-2c7a-4ee9-8fb8-a48bb13ec435" />


Video Demo:
https://drive.google.com/file/d/1GRx4i2sXR0ASKQf_AOxu0R45blrvTqTB/view?usp=sharing
