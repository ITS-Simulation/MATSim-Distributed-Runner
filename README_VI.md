[![Generate Workers](https://github.com/ITS-Simulation/MATSim_Distributed_Runner/actions/workflows/sync-config.yml/badge.svg)](https://github.com/ITS-Simulation/MATSim_Distributed_Runner/actions/workflows/sync-config.yml)

# MATSim Distributed Runner

[🇬🇧 English](./README.md)

Repo này đóng vai trò trung tâm điều phối cho hệ thống mô phỏng MATSim phân tán. Nó tự động hóa việc quản lý và phân phối cấu hình cho các máy trạm (worker) trên nhiều nền tảng phần cứng khác nhau.

## 🚀 Cơ Chế Tự Động Hóa Nhánh

Dự án này sử dụng mô hình quản lý nhánh tự động 100%. 
**Lưu ý quan trọng: Vui lòng không chỉnh sửa thủ công các nhánh runner.**

*   **`main`**: Nhánh tham chiếu của dự án (source of truth). Nhánh này chứa file cấu hình trung tâm `config.yaml`, `Dockerfile` gốc, và file mẫu `docker-compose.yaml`.
*   **Các Nhánh Runner** (ví dụ: `i7`, `i7-high`, `i5`): Đây là các nhánh được hệ thống tự động sinh ra để đại diện cho từng cấu hình phần cứng và số lượng worker cụ thể.

### Cách Hệ Thống Hoạt Động
1.  **Cấu hình phần cứng**: Giới hạn tài nguyên (CPU/RAM) và số lượng worker được thiết lập tập trung trong file `config.yaml`.
2.  **Tự động đồng bộ**: Bất cứ khi nào nhánh `main` có sự thay đổi, một GitHub Action (`sync-config.yml`) sẽ tự động chạy và thực hiện các việc sau:
    *   Tạo mới hoặc cập nhật các nhánh runner dựa trên cấu hình đã lưu.
    *   Tự động điền các thông số giới hạn CPU/RAM và số lượng worker vào file `docker-compose.yaml` của từng nhánh tương ứng.
    *   Đồng bộ file `Dockerfile` mới nhất từ nhánh `main` sang tất cả các nhánh con.
    *   Tạo sẵn file `README.md` và `README_VI.md` song ngữ trên mỗi nhánh runner, ghi rõ chi tiết về phần cứng, phiên bản, và thông tin kịch bản đang chạy.
    *   Dọn dẹp hệ thống bằng cách xóa đi các nhánh cũ không còn được khai báo trong `config.yaml`.

## ⚙️ Cấu hình hệ thống (`config.yaml`)

Các profile cấu hình máy chạy (runner) được thiết lập trong file `config.yaml` trên nhánh `main`. Dưới đây là một ví dụ:

```yaml
ip: "192.168.1.1"  # Địa chỉ IP của máy chủ trung tâm

runner:
  i7:              # Tên Profile (có thể đặt tùy ý, miễn là hợp lệ chuẩn git nhánh)
    hw:
      cpu: 26.0    # Giới hạn CPU dành cho Docker
      memory: "10G" # Giới hạn RAM dành cho Docker
    workers:
      high: 10     # Hệ thống sẽ tạo nhánh 'i7-high' với 10 worker
      normal: 8    # Hệ thống sẽ tạo nhánh 'i7' với 8 worker (đây đóng vai trò là nhánh cơ sở)
      mid: 6       # Hệ thống sẽ tạo nhánh 'i7-mid' với 6 worker
      low: 4       # Hệ thống sẽ tạo nhánh 'i7-low' với 4 worker
  i5:
    hw:
      cpu: 18.0
      memory: "5G"
    workers:
      high: 6
      normal: 4
```

### Quy Tắc Đặt Tên
*   **Hoàn toàn linh hoạt**: Tên profile (như `i7`, `i5`) hay tên phân khúc/tier (như `high`, `low`) không bị bó buộc. Có thể áp dụng bất kỳ cách đặt tên nào phù hợp với môi trường triển khai thực tế (chẳng hạn như đặt theo dòng CPU, tên phòng ban).
*   **Nhánh Cơ Sở**: Hai từ khóa phân khúc `normal` và `base` là những từ khóa đặc biệt. Khi một trong hai từ này được sử dụng, hệ thống sẽ tạo ra một **nhánh cơ sở** mang tên trùng khớp với profile (ví dụ: nhánh `i7`). Bất kỳ từ khóa nào khác sẽ tạo ra nhánh với định dạng `<profile>-<tier>` (ví dụ: `i7-high`). Nếu cả `normal` lẫn `base` đều không được khai báo, profile đó sẽ không có nhánh cơ sở.
*   **Quy tắc của Git**: Do những cái tên này sẽ trực tiếp trở thành tên nhánh trên Git, chúng cần phải tuân thủ nghiêm ngặt quy tắc đặt tên nhánh chuẩn. Điều này có nghĩa là không được chứa dấu cách, không dùng các ký tự đặc biệt như `~`, `^`, `:`, `?`, `*`, `[`, hoặc `\`. Tên cũng không được bắt đầu hay kết thúc bằng dấu `.` hoặc `/`, không chứa hai dấu chấm liên tiếp `..`, và không được có đuôi `.lock`.

## 🛠️ Triển khai

Để triển khai một cấu hình runner cụ thể, chỉ cần pull nhánh tương ứng:

```bash
# Triển khai cấu hình i7 hiệu năng cao (high-performance)
git clone https://github.com/ITS-Simulation/MATSim_Distributed_Runner.git
git checkout i7-high
docker compose up -d --build
```

## 📦 Quy trình Cập nhật

Các bản cập nhật được kích hoạt tự động từ repository [`MATSim-Bus-Optimizer`](https://github.com/ITS-Simulation/MATSim-Bus-Optimizer):
1.  Release mới trong `MATSim-Bus-Optimizer` → Cập nhật `Dockerfile` trên nhánh `main` (phiên bản, checksum).
2.  Quy trình `sync-config` kích hoạt → Cập nhật tất cả các nhánh runner.
3.  Các runner chỉ cần pull về và khởi động lại.
