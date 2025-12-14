# Laporan IDS KELOMPOK 7

**Mata Kuliah:** Keamanan Jaringan Komputer  
**Kelas:** A
**Anggota Kelompok:**
1. **Nadia Kirana Afifah Prahandita** - [5027241005]
2. **Tiara Putri Prasetya** - [5027241013]
3. **Zahra Khaalishah** - [5027241070]
4. **Mutiara Diva Jaladitha** - [5027241083]
---

## 1. Pendahuluan
Proyek ini bertujuan untuk mensimulasikan dan mengimplementasikan **Intrusion Detection System (IDS)** menggunakan **Suricata** pada lingkungan jaringan virtual (GNS3). Sistem dikonfigurasi menggunakan mode *Inline* (Bridge) untuk memantau dan mendeteksi lalu lintas mencurigakan yang menargetkan server riset.

Skenario serangan yang diujikan mencakup:
1. **Port Scanning** (Reconnaissance)
2. **SSH Brute Force** (Intrusion Attempt)
3. **HTTP Data Exfiltration** (Data Theft)

---

## 2. Topologi Jaringan
Simulasi dilakukan menggunakan topologi sebagai berikut:

<img width="615" height="736" alt="Screenshot 2025-12-14 071609" src="https://github.com/user-attachments/assets/6ea407f2-b527-41a0-9932-83dd5bbad1e4" />


### Penjelasan Topologi:
* **Edge Router:** Gateway utama yang menghubungkan jaringan internal ke internet (NAT).
* **Switch1 (Core):** Titik distribusi utama ke berbagai subnet (Admin, Student, Akademik, Riset, Guest).
* **Posisi IDS (ids-ubuntu):** IDS ditempatkan secara **Inline (Bridge Mode)** di antara **Switch1** dan **Student-Router**.
  * **Alasan:** Posisi ini strategis untuk mengisolasi jaringan Student. Seluruh lalu lintas dari mahasiswa yang menuju jaringan lain (seperti Server Riset) **wajib** melewati inspeksi IDS, sehingga tidak ada serangan yang lolos tanpa terdeteksi.

---

## 3. Konfigurasi Sistem

### A. Konfigurasi Bridge (Transparan)
Agar IDS dapat bekerja di tengah jaringan tanpa mengubah skema IP (Layer 2 Transparent), kami menggabungkan interface `eth0` dan `eth1` menjadi jembatan (`br0`).

```bash
brctl addbr br0         # Membuat interface bridge
brctl addif br0 eth0    # Menambahkan eth0 ke bridge
brctl addif br0 eth1    # Menambahkan eth1 ke bridge
ip link set br0 up      # Mengaktifkan bridge
```
### B. Suricata Rules (Aturan Deteksi)
Kami menambahkan aturan khusus (custom rules) pada file /etc/suricata/rules/local.rules untuk mendeteksi serangan spesifik dari IP Student ke Server Riset (10.20.30.10).

1. Deteksi Port Scanning (Nmap)
Mendeteksi pemindaian paket TCP SYN pada port kritis (22, 80, 443).

```
alert tcp any any -> any [22,80,443] (msg:"ALERT: Nmap SYN Scan Deteksi"; flags:S; sid:100001; rev:3;)
```

2. Deteksi SSH Brute Force
Mendeteksi percobaan koneksi paksa ke layanan SSH server target.

```
alert tcp any any -> 10.20.30.10 22 (msg:"ALERT: Percobaan SSH Brute Force ke Server Riset"; flags:S; sid:100002; rev:3;)
```

3. Deteksi Eksfiltrasi Data (HTTP)
Mendeteksi upaya pengunduhan file sensitif melalui protokol HTTP.

```
alert tcp any any -> 10.20.30.10 80 (msg:"ALERT: Percobaan Download File HTTP dari Server Riset"; flags:S; sid:100003; rev:4;)
```

4. Hasil Simulasi dan Analisis Log
Pengujian dilakukan dengan melancarkan serangan dari Student Host (10.20.20.10) menuju Riset Host (10.20.30.10).
Berikut adalah tangkapan layar fast.log pada IDS saat serangan berlangsung:
![WhatsApp Image 2025-12-13 at 3 03 20 PM](https://github.com/user-attachments/assets/4f883bd9-dd68-4013-b9d3-23437dfaac10)


Analisis Log:
#### 1. Nmap SYN Scan (sid:100001)
- Terlihat pada baris log: ALERT: Nmap SYN Scan Deteksi.
- Analisis: IDS berhasil menangkap pola paket SYN beruntun yang dikirimkan oleh Nmap untuk memindai port terbuka.

#### 2. SSH Brute Force (sid:100002)
- Terlihat pada baris log: ALERT: Percobaan SSH Brute Force.
- Analisis: IDS mendeteksi inisiasi koneksi ke port 22. Log ini membuktikan bahwa IDS mampu memberikan peringatan dini (Early Warning) terhadap upaya intrusi ilegal.

### 5. Kesimpulan
Berdasarkan hasil praktikum, dapat disimpulkan bahwa:
- Topologi Inline Bridge efektif untuk melakukan inspeksi paket secara transparan tanpa mengganggu konektivitas jaringan.
- Suricata IDS berhasil mendeteksi ancaman sesuai dengan signature rules yang didefinisikan, memberikan visibilitas penuh terhadap lalu lintas jaringan yang mencurigakan.
- Sistem ini siap dikembangkan lebih lanjut menjadi IPS (Intrusion Prevention System) untuk memblokir serangan secara otomatis.
