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
RECEIVER_OPTIONS="--device 0 --device-type rtlsdr --gain auto --ppm 0"
DECODER_OPTIONS="--max-range 450 --write-json-every 1 --modeac"
NET_OPTIONS="--net --net-ri-port 30001 --net-ro-port 30002 --net-sbs-port 30003 --net-bi-port 30004,30104 --net-bo-port 30005"
JSON_OPTIONS="--json-location-accuracy 2 --range-outline-hours 24 --lat 21.024587 --lon 105.773481"
```
