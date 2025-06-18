# Mining-Verus-GPU
Berikut adalah script mining **VerusCoin (VRSC)** untuk **Ubuntu 22.04/24.04** menggunakan GPU **NVIDIA** dengan **T-Rex Miner** dan **lolMiner** (format `.sh`).  

---

### **1. Persyaratan Awal (Install Driver & Dependensi)**
Pastikan driver NVIDIA dan CUDA terinstall:
```bash
# Hapus driver lama (jika ada)
sudo apt purge *nvidia* *cuda* -y  
sudo apt autoremove -y  

# Install driver NVIDIA (versi terbaru)
sudo ubuntu-drivers autoinstall  
sudo apt install nvidia-cuda-toolkit -y  

# Reboot
sudo reboot
```
Setelah reboot, verifikasi instalasi:
```bash
nvidia-smi  # Cek status GPU
```

---

### **2. Script Mining untuk T-Rex Miner (NVIDIA)**
**Unduh T-Rex Miner**:
```bash
wget https://github.com/trexminer/T-Rex/releases/download/0.26.8/t-rex-0.26.8-linux.tar.gz  
tar -xvf t-rex-0.26.8-linux.tar.gz  
cd t-rex-0.26.8-linux
```
**Buat Script `mine_verus.sh`**:
```bash
#!/bin/bash  
./t-rex -a verushash \  
  -o stratum+tcp://eu.luckpool.net:3956 \  
  -u YOUR_VERUS_WALLET_ADDRESS.RIG_NAME \  
  -p x \  
  --log-path /var/log/trex.log \  
  --pl 70      # Batas daya 70%  
```
**Jalankan**:
```bash
chmod +x mine_verus.sh  
./mine_verus.sh  
```
**Opsional (Overclocking)**:
Tambahkan flag berikut ke script:
```bash
--lock-cclock 1200 \  # Lock core clock  
--mclock 1000 \      # Memory clock offset  
--fan 75             # Kontrol kipas  
```

---

### **3. Script Mining untuk lolMiner (NVIDIA/AMD)**
**Unduh lolMiner**:
```bash
wget https://github.com/Lolliedieb/lolMiner-releases/releases/download/1.82/lolMiner_v1.82_Lin64.tar.gz  
tar -xvf lolMiner_v1.82_Lin64.tar.gz  
cd 1.82
```
**Buat Script `mine_verus_lol.sh`**:
```bash
#!/bin/bash  
./lolMiner --algo VERUSHASH \  
  --pool eu.luckpool.net:3956 \  
  --user YOUR_VERUS_WALLET_ADDRESS.RIG_NAME \  
  --pass x \  
  --tls 0 \  
  --logfile /var/log/lolminer.log \  
  --cclock +100 \    # Core clock offset  
  --mclock +800      # Memory clock offset  
```
**Jalankan**:
```bash
chmod +x mine_verus_lol.sh  
./mine_verus_lol.sh  
```

---

### **4. Auto-Start Mining (Systemd Service)**
Agar mining berjalan otomatis saat boot, buat service:
```bash
sudo nano /etc/systemd/system/verus_miner.service
```
**Isi File** (contoh untuk T-Rex):
```ini
[Unit]  
Description=VerusCoin Miner (T-Rex)  
After=network.target  

[Service]  
User=$(whoami)  
WorkingDirectory=/path/to/t-rex/folder  
ExecStart=/bin/bash /path/to/mine_verus.sh  
Restart=always  

[Install]  
WantedBy=multi-user.target  
```
**Aktifkan**:
```bash
sudo systemctl enable verus_miner.service  
sudo systemctl start verus_miner.service  
```
Cek status:
```bash
sudo systemctl status verus_miner.service  
```

---

### **5. Tips Optimasi di Ubuntu**
1. **Matikan GUI (Opsional)**:  
   Jika mining di headless server, matikan X11 untuk menghemat RAM/GPU:
   ```bash
   sudo systemctl set-default multi-user.target  
   sudo reboot  
   ```
2. **Overclock dengan `nvidia-settings`**:
   ```bash
   sudo nvidia-smi -pm 1          # Enable persistence mode  
   sudo nvidia-smi -pl 150        # Batas daya 150W  
   nvidia-settings -a '[gpu:0]/GPUGraphicsClockOffset[3]=100'  
   nvidia-settings -a '[gpu:0]/GPUMemoryTransferRateOffset[3]=800'  
   ```
3. **Cek Suhu**:
   ```bash
   watch -n 1 nvidia-smi  
   ```

---

### **Catatan Penting**
- Ganti `YOUR_VERUS_WALLET_ADDRESS` dengan alamat Verus Anda (contoh: `R9v....XXX`).  
- Pool alternatif: `sg.luckpool.net:3956` (Asia), `na.luckpool.net:3956` (Amerika).  
- Untuk **monitoring remote**, gunakan `htop`, atau `nvtop`

Jika ada error, cek log (`/var/log/trex.log` atau `journalctl -u verus_miner.service -f`).
