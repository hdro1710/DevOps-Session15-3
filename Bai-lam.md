## 3. Yêu cầu của bài tập

### 3.1. Chỉnh sửa Alerting Rule

Tạo file `alert_rules.yml` trong thư mục dự án Prometheus và khai báo rule `HighCPUUsage` với nội dung:

```yaml
groups:
  - name: CPU_Alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100) > 85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage detected"
```

### 3.2. Điều chỉnh ngưỡng cảnh báo

Thay đổi điều kiện trong biểu thức PromQL từ:

```promql
... > 20
```

thành:

```promql
... > 85
```

Việc nâng ngưỡng lên 85% giúp hạn chế cảnh báo khi CPU chỉ sử dụng ở mức tải thông thường.

### 3.3. Điều chỉnh thời gian theo dõi

Thay đổi thời gian duy trì điều kiện cảnh báo từ:

```yaml
for: 10s
```

thành:

```yaml
for: 5m
```

Prometheus chỉ chuyển cảnh báo sang trạng thái `Firing` khi điều kiện CPU vượt 85% được duy trì liên tục trong 5 phút. Điều này giúp hạn chế cảnh báo do những đợt tăng tải CPU ngắn hạn.

### 3.4. Kiểm tra cú pháp bằng promtool

Sử dụng lệnh:

```bash
sudo docker cp ~/prometheus/alert_rules.yml prometheus:/tmp/alert_rules.yml

sudo docker exec prometheus promtool check rules /tmp/alert_rules.yml
```

Kết quả:

```text
Checking /tmp/alert_rules.yml
  SUCCESS: 1 rules found
```

Kết quả xác nhận file rule hợp lệ và Prometheus Tool đã tìm thấy một alerting rule.

### 3.5. Khai báo và nạp Alerting Rules

Bổ sung vào file `prometheus.yml`:

```yaml
rule_files:
  - "alert_rules.yml"
```

Đồng thời, bổ sung volume mount trong `compose.yaml`:

```yaml
- ./alert_rules.yml:/etc/prometheus/alert_rules.yml:ro
```

Sau khi kiểm tra cấu hình Docker Compose, cập nhật service Prometheus để áp dụng cấu hình mới.

## 4. Kết quả mong muốn và xác minh

### 4.1. Kết quả đạt được

- Đã thay đổi ngưỡng cảnh báo CPU từ 20% lên 85%.
- Đã tăng thời gian duy trì điều kiện cảnh báo từ 10 giây lên 5 phút.
- Đã kiểm tra cú pháp thành công bằng `promtool`.
- Đã cấu hình Prometheus đọc file `alert_rules.yml`.
- Đã mount file rule vào container Prometheus thành công.

### 4.2. Xác minh qua Prometheus API

Sử dụng lệnh:

```bash
sudo docker exec prometheus wget -qO- http://localhost:9090/api/v1/rules
```

Kết quả trả về xác nhận:

| Thuộc tính | Giá trị |
|---|---|
| Rule | `HighCPUUsage` |
| Group | `CPU_Alerts` |
| Threshold | `> 85` |
| Duration | `300` giây (5 phút) |
| Severity | `warning` |
| State | `inactive` |
| Health | `ok` |

Trạng thái `inactive` cho biết điều kiện cảnh báo hiện chưa được thỏa mãn. Rule đã được Prometheus nạp thành công và đang hoạt động bình thường.

### 4.3. Kết luận

Bài thực hành đã hoàn thành các yêu cầu về chỉnh sửa, kiểm tra cú pháp và triển khai Prometheus Alerting Rules.

Việc nâng ngưỡng cảnh báo lên 85% và tăng thời gian duy trì lên 5 phút giúp hạn chế các cảnh báo không cần thiết do những biến động CPU ngắn hạn. Hệ thống chỉ chuyển cảnh báo sang trạng thái `Firing` khi CPU vượt ngưỡng 85% liên tục trong 5 phút.

Lưu ý: Bài thực hành đã xác minh rule được nạp thành công, nhưng chưa thực hiện mô phỏng tải CPU để kiểm tra trực tiếp quá trình chuyển trạng thái sang `Firing`.
