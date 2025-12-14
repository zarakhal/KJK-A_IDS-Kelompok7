# Laporan IDS KELOMPOK 7

**Mata Kuliah:** Keamanan Jaringan Komputer  
**Kelas:** A
**Anggota Kelompok:**
1. **[NAMA LENGKAP ANGGOTA 1]** - [NRP]
2. **[NAMA LENGKAP ANGGOTA 2]** - [NRP]
3. **[NAMA LENGKAP ANGGOTA 3]** - [NRP]

---

## 1. Pendahuluan
Proyek ini bertujuan untuk mensimulasikan dan mengimplementasikan **Intrusion Detection System (IDS)** menggunakan **Suricata** pada lingkungan jaringan virtual (GNS3). Sistem dikonfigurasi menggunakan mode *Inline* (Bridge) untuk memantau dan mendeteksi lalu lintas mencurigakan yang menargetkan server riset.

Skenario serangan yang diujikan mencakup:
1. **Port Scanning** (Reconnaissance)
2. **SSH Brute Force** (Intrusion Attempt)
3. **HTTP Data Exfiltration** (Data Theft)

---

## 2. Topologi Jaringan
Simulasi dilakukan menggunakan topologi hierarki sebagai berikut:

![Topologi GNS3](<img width="615" height="736" alt="image" src="https://github.com/user-attachments/assets/fa5db7e0-0aef-4683-9fe8-00e304bd10b7" />)
*(Gambar 1: Topologi Implementasi IDS di GNS3*

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
# Perintah konfigurasi di IDS Ubuntu
brctl addbr br0         # Membuat interface bridge
brctl addif br0 eth0    # Menambahkan eth0 ke bridge
brctl addif br0 eth1    # Menambahkan eth1 ke bridge
ip link set br0 up      # Mengaktifkan bridge
