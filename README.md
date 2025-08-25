# mlat_deployments
# mlat_client 
Bước 1: Cài đặt image cho Pi 4 bằng công cụ Pi image và chọn bản Legacy cũ nhất (Do bản mới không tương thích một số chỗ)

Bước 2: Cài đặt VPN tailscale
```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```
Bước 3: Cài đặt readsb trên Pi 4
```bash
sudo bash -c "$(wget -O - https://github.com/wiedehopf/adsb-scripts/raw/master/readsb-install.sh)"
sudo reboot
```
Sau đó cài lighttpd để hiển thị giao diện web tar1090
```bash
sudo apt install -y lighttpd
```
Sau đó vào file config của readsb để enable modeac, và điền lat, long của trạm thu theo mẫu sau

```bash
sudo nano /etc/default/readsb
```

```bash
RECEIVER_OPTIONS="--device 0 --device-type rtlsdr --gain auto --ppm 0"
DECODER_OPTIONS="--max-range 450 --write-json-every 1 --modeac"
NET_OPTIONS="--net --net-ri-port 30001 --net-ro-port 30002 --net-sbs-port 30003 --net-bi-port 30004,30104 --net-bo-port 30005"
JSON_OPTIONS="--json-location-accuracy 2 --range-outline-hours 24 --lat 21.024587 --lon 105.773481"
```
Sau đó khởi động lại readsb 
```bash
sudo systemctl daemon-reload
sudo systemctl restart readsb.service
```
Bước 4: Cài đặt mlat-client
Link tại https://github.com/adsb-related-code/mlat-client
Đầu tiên phải tạo virtual environment cho mlat-client và kích hoạt nó
```bash
python3 -m venv mlat-client
source mlat-client/bin/activate
```
Tiếp theo, tạo một thư mục để chứa repo mlat-client, tránh clone vào thư mục chứa venv. Sau đó clone code về
```bash
mkdir mlat-client-dir
cd mlat-client-dir
git clone https://github.com/adsb-related-code/mlat-client
cd mlat-client/
```
Khi này chúng ta nên ở thư mục /home/username/mlat-client-dir/mlat-client và cài đặt mlat-client bằng lệnh sau
```bash
python3 setup.py build
python3 setup.py install
```
Check thấy ở trong /home/mlat-client-5/mlat-client/bin có mlat-client là được. Test thử bằng cách invoke 
```bash
mlat-client --help
```
mlat-client trả lại hướng dẫn mà không gặp lỗi gì là được

Sau bước này nên deactivate venv trước đó đi 
```bash
deactivate
```


Bước 5: Viết file systemd cho mlat-client để service tự chạy và khởi động lại khi có lỗi

Tạo mới file service
```bash
sudo nano /etc/systemd/system/mlat-client.service
```
Sau đó copy nội dung của file service vào, lưu ý phải sửa user, đường dẫn tới mlat-client, lat, long của client cho đúng!!!
```bash[Unit]
Description=mlat-client Service
After=network.target

[Service]
User=username
ExecStart=/home/mlat-client-1/mlat-client/mlat-client-venv/bin/mlat-client \
  --input-type dump1090 \
  --input-connect 127.0.0.1:30005 \
  --lat 21.024587 \
  --lon 105.773481 \
  --alt 28 \
  --user ckt-client \
  --server 100.107.147.47:31090 \
  --no-udp \
  --results beast,connect,127.0.0.1:30104 \
  --results beast,connect,100.107.147.47:30004
Restart=always

[Install]
WantedBy=multi-user.target
```
Sau đó reload lại và restart lại mlat-client service
```bash
sudo systemctl daemon-reload
sudo systemctl restart mlat-client.service
```
Bước 6: Tạo service socat để forward dữ liệu beast sang địa chỉ máy hiển thị

Install socat
```bash
sudo apt-get install socat
```
Tạo script chuyển tiếp
```bash
sudo nano /usr/local/bin/socat-beast-forward.sh
```
Copy nội dung vào file script
```bash
#!/bin/bash
# Chuyển tiếp dữ liệu Beast từ dump1090-fa local tới máy A

# Địa chỉ IP máy net-only (thay đổi nếu cần)
TARGET_IP="100.124.196.54"

# Chuyển tiếp từ cổng local 30005 đến máy net-only
exec socat -u TCP:127.0.0.1:30005 TCP:$TARGET_IP:30004
```

Sau đó cấp quyền thực thi:
```bash
sudo chmod +x /usr/local/bin/socat-beast-forward.sh
```
Sau đó tạo file systemd để service chạy
```bash
sudo nano /etc/systemd/system/socat-beast.service
```
Copy nội dung file service socat như sau
```bash
[Unit]
Description=Forward dump1090-fa Beast data to net-only receiver
After=network.target dump1090-fa.service
Requires=dump1090-fa.service

[Service]
ExecStart=/usr/local/bin/socat-beast-forward.sh
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```
Và khởi động lại dịch vụ
```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable socat-beast.service
sudo systemctl start socat-beast.service
```
Bước 7: Optional (nên cài để theo dõi tình hình hệ thống). Cài graphs1090

Chạy lệnh sau
```bash
sudo bash -c "$(curl -L -o - https://github.com/wiedehopf/graphs1090/raw/master/install.sh)"
```
