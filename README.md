# Đề 3 - Website quảng bá sản phẩm (WordPress)
Hệ thống: WordPress, MySQL, phpMyAdmin, Nginx, Prometheus, Grafana, Loki.
## Chạy
1. cp .env.example .env và điền mật khẩu
2. Sinh chứng chỉ tự ký vào nginx/certs
3. docker compose up -d
## Dừng
docker compose down   (down -v sẽ xóa dữ liệu)
