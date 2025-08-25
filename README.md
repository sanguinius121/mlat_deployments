# mlat_deployments
# mlat_client 
Bước 1: Cài đặt image cho Pi 4 bằng công cụ Pi image và chọn bản Legacy cũ nhất (Do bản mới không tương thích một số chỗ)
Bước 2: Cài đặt readsb trên Pi 4
```bash
sudo bash -c "$(wget -O - https://github.com/wiedehopf/adsb-scripts/raw/master/readsb-install.sh)"
sudo reboot
```
Sau đó cài lighttpd để hiển thị giao diện web tar1090
```bash
sudo apt install -y lighttpd
```
