# Nhận dạng giọng nói tiếng Việt bằng TinyML trên ESP32

## 1. Giới thiệu

Dự án xây dựng hệ thống nhận dạng lệnh giọng nói tiếng Việt sử dụng **TinyML trên ESP32-WROOM-32E**. Hệ thống thu âm thông qua microphone INMP441, trích xuất đặc trưng âm thanh bằng MFCC và sử dụng mô hình CNN để phân loại lệnh.

Mục tiêu của dự án là nhận dạng các lệnh bật và tắt đèn mà không cần gửi dữ liệu âm thanh lên máy chủ hoặc sử dụng kết nối Internet trong quá trình suy luận.

## 2. Chức năng chính

- Thu âm giọng nói tiếng Việt bằng microphone INMP441.
- Tiền xử lý tín hiệu âm thanh và trích xuất đặc trưng MFCC.
- Sử dụng mô hình CNN để phân loại lệnh giọng nói.
- Nhận dạng lệnh bật đèn và tắt đèn.
- Phân loại các âm thanh không thuộc lệnh điều khiển thành `UNKNOWN` hoặc `NOISE`.
- Thực hiện suy luận trực tiếp trên ESP32.
- Điều khiển LED dựa trên kết quả nhận dạng.

## 3. Kiến trúc hệ thống

Quy trình hoạt động của hệ thống:

```text
Giọng nói
    │
    ▼
Microphone INMP441
    │
    ▼
Thu thập tín hiệu âm thanh
    │
    ▼
Tiền xử lý âm thanh
    │
    ▼
Trích xuất đặc trưng MFCC
    │
    ▼
Mô hình CNN
    │
    ▼
Phân loại lệnh giọng nói
    │
    ├── ON_LIGHT  ──► Bật LED
    │
    ├── OFF_LIGHT ──► Tắt LED
    │
    ├── UNKNOWN   ──► Không điều khiển
    │
    └── NOISE     ──► Không điều khiển
```

Mô hình được huấn luyện trên máy tính và triển khai lên ESP32 để thực hiện suy luận tại thiết bị.

## 4. Phần cứng sử dụng

| STT | Linh kiện | Chức năng |
|---|---|---|
| 1 | ESP32-WROOM-32E | Vi điều khiển thực hiện suy luận TinyML |
| 2 | INMP441 | Microphone kỹ thuật số thu âm thanh |
| 3 | LED | Mô phỏng thiết bị cần điều khiển |
| 4 | Điện trở 220–330 Ω | Hạn chế dòng điện qua LED |
| 5 | Nút nhấn | Kích hoạt thu âm nếu sử dụng chế độ nhấn để nói |
| 6 | Breadboard và dây nối | Kết nối phần cứng |
| 7 | Cáp USB | Cấp nguồn và nạp chương trình |

## 5. Công cụ và công nghệ

- **Arduino IDE:** Biên dịch và nạp chương trình cho ESP32.
- **Edge Impulse:** Xây dựng pipeline xử lý âm thanh, huấn luyện và đánh giá mô hình.
- **Python:** Hỗ trợ xử lý dữ liệu và huấn luyện mô hình khi cần.
- **MFCC:** Trích xuất đặc trưng âm thanh.
- **CNN:** Phân loại các lệnh giọng nói.
- **TensorFlow Lite for Microcontrollers / thư viện triển khai tương ứng:** Hỗ trợ triển khai mô hình TinyML trên vi điều khiển.

## 6. Các lớp nhận dạng

Mô hình được thiết kế với bốn lớp phân loại.

| Nhãn | Ví dụ | Ý nghĩa |
|---|---|---|
| `ON_LIGHT` | Bật đèn, Mở đèn, Cho đèn sáng | Lệnh bật đèn |
| `OFF_LIGHT` | Tắt đèn, Đóng đèn, Cho đèn tắt | Lệnh tắt đèn |
| `UNKNOWN` | Các câu nói không thuộc nhóm lệnh mục tiêu | Âm thanh hoặc lời nói ngoài các lệnh đã định nghĩa |
| `NOISE` | Tiếng quạt, tiếng môi trường và các âm thanh không phải lệnh | Nhiễu âm thanh |

Việc bổ sung hai lớp `UNKNOWN` và `NOISE` nhằm giúp mô hình phân biệt lệnh điều khiển với các âm thanh không mong muốn. Hiệu quả thực tế phụ thuộc vào dữ liệu huấn luyện và cách thiết lập ngưỡng nhận dạng.

## 7. Cấu hình mô hình AI

Các thông số dưới đây được tổng hợp từ cấu hình mô hình trên Edge Impulse.

### Tiền xử lý âm thanh

| Thông số | Giá trị |
|---|---|
| Tần số lấy mẫu | 16.000 Hz |
| Thời lượng cửa sổ âm thanh | 2.500 ms |
| Mức tăng cửa sổ | 500 ms |
| Phương pháp trích xuất đặc trưng | MFCC |
| Số hệ số MFCC | 15 |
| Độ dài khung (Frame length) | 0,025 s |
| Bước dịch khung (Frame stride) | 0,02 s |
| Số bộ lọc | 32 |
| Kích thước FFT | 512 |
| Tần số thấp | 80 Hz |
| Tần số cao | 4.000 Hz |
| Hệ số Pre-emphasis | 0,98 |

### Huấn luyện mô hình

| Thông số | Giá trị |
|---|---|
| Kiến trúc mô hình | CNN |
| Số lớp phân loại | 4 |
| Số chu kỳ huấn luyện | 50 |
| Learning rate | 0,005 |

Các thông số có thể được điều chỉnh khi mở rộng tập dữ liệu hoặc tối ưu mô hình.

## 8. Sơ đồ kết nối

### ESP32-WROOM-32E và INMP441

| INMP441 | ESP32-WROOM-32E |
|---|---|
| VDD | 3V3 |
| GND | GND |
| SCK / BCLK | GPIO14 |
| WS / LRCL | GPIO15 |
| SD | GPIO32 |
| L/R | GND |

### Các chân điều khiển

| Thành phần | Chân ESP32 | Ghi chú |
|---|---|---|
| Nút nhấn | GPIO25 | Nối nút nhấn giữa GPIO25 và GND nếu sử dụng `INPUT_PULLUP` |
| LED báo thu âm | GPIO2 | LED báo trạng thái, nếu được sử dụng |
| LED điều khiển | GPIO26 | Nối LED qua điện trở 220–330 Ω |

**Lưu ý:** Sơ đồ trên mô tả cấu hình chân dự kiến của dự án. Cần đối chiếu với chương trình firmware thực tế trước khi đấu nối. Không kết nối LED trực tiếp mà không có điện trở hạn dòng.

## 9. Kết quả đánh giá mô hình

Theo các kết quả đánh giá được ghi nhận trên Edge Impulse, mô hình đạt các chỉ số sau:

### Kết quả Validation

| Chỉ số | Giá trị |
|---|---:|
| Validation accuracy | 91,2% |
| Validation loss | 0,28 |
| AUC | 0,99 |

### Kết quả Model Testing

| Chỉ số | Giá trị |
|---|---:|
| Accuracy | 87,20% |
| Weighted precision | 0,93 |
| Weighted recall | 0,92 |
| Weighted F1-score | 0,92 |
| AUC | 1,00 |

### Tài nguyên mô hình

| Thông số | Giá trị |
|---|---:|
| Inference time | 110 ms |
| Peak RAM | 20,8 KB |
| Flash sử dụng | 38,0 KB |

Các chỉ số trên là kết quả được báo cáo từ quá trình đánh giá mô hình và ước lượng tài nguyên trên Edge Impulse. Chúng không mặc nhiên đại diện cho độ chính xác thực tế trong mọi môi trường hoặc hiệu năng đo trực tiếp trên phần cứng cuối cùng.

Kết quả nhận dạng có thể thay đổi tùy thuộc vào giọng nói, khoảng cách đến microphone, tiếng ồn môi trường và chất lượng dữ liệu huấn luyện.

## 10. Cấu trúc repository

Cấu trúc thư mục có thể được tổ chức như sau:

```text
tinyml-vietnamese-voice-recognition/
│
├── README.md
├── data/
│   └── README.md
├── model/
│   └── README.md
├── firmware/
│   └── README.md
└── docs/
    └── README.md
```

> Lưu ý: Đây là cấu trúc tham khảo. Hãy điều chỉnh theo các thư mục và tệp thực tế trong repository. Không cần tạo các thư mục trống nếu dự án chưa sử dụng chúng.

