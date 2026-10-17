# Đề 3: Website quảng bá sản phẩm (WordPress)

Hệ thống chạy bằng Docker Compose trên Ubuntu Server:
- WordPress + MySQL 8.0 + phpMyAdmin
- Nginx reverse proxy, HTTPS chứng chỉ tự ký, security headers
- Prometheus + Grafana (giám sát), Loki + Promtail (log tập trung)
- Hai network: `app-network` (ứng dụng) và `monitoring` (giám sát)

## Yêu cầu
Ubuntu Server, Docker, Docker Compose v2.

## Cài đặt
    cp .env.example .env    # rồi điền mật khẩu và đặt SERVER_IP đúng IP máy chủ trong .env
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
      -keyout nginx/certs/selfsigned.key -out nginx/certs/selfsigned.crt \
      -subj "/C=VN/O=Lab/CN=<SERVER_IP>"
    docker compose up -d

Sau khi MySQL chạy lần đầu, tạo user `exporter` cho mysql-exporter (mật khẩu lấy từ `EXPORTER_PASSWORD` trong `.env`):

    set -a; . ./.env; set +a
    docker exec -i mysql mysql -uroot -p"$MYSQL_ROOT_PASSWORD" -e "CREATE USER IF NOT EXISTS 'exporter'@'%' IDENTIFIED BY '$EXPORTER_PASSWORD'; GRANT SELECT, PROCESS, REPLICATION CLIENT ON *.* TO 'exporter'@'%'; FLUSH PRIVILEGES;"
    docker compose restart mysql-exporter

## Địa chỉ truy cập
| Dịch vụ | Địa chỉ |
|---|---|
| Website | https://\<SERVER_IP\> |
| phpMyAdmin | http://\<SERVER_IP\>:8082 |
| Prometheus | http://\<SERVER_IP\>:9090 |
| Grafana | http://\<SERVER_IP\>:3000 |

## Dừng hệ thống
- `docker compose down`: dừng, giữ dữ liệu
- `docker compose down -v`: dừng và xóa dữ liệu

Nếu gặp 502 Bad Gateway sau khi tạo lại container: `docker compose restart nginx`.

## Bảo mật
- Website chỉ qua Nginx (80/443), MySQL không publish cổng ra host
- UFW mặc định chặn kết nối đến (deny incoming)
- Container chạy với `no-new-privileges` (trừ cadvisor và promtail)
- Mật khẩu nằm trong `.env`, không đưa lên Git
- Cổng phpMyAdmin (8082), Prometheus (9090), Grafana (3000) publish ra host cho mục đích lab
