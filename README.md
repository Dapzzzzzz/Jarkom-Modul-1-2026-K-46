# Jarkom K-46

| 	Nama              	| NRP        |
| ---------------------	| ---------- |
| Hendra Manudinata 	| 5027251051 |
| Daffa Rifqi As Shidiq	| 5027251038 |

* Kelompok: K-46

* Prefix IP: 192.234.x.x

---

# Script Tambahan

Untuk memudahkan pengunduhan file soal dari Google Drive langsung ke Node, kami membuat shell script sederhana untuk melakukan *direct download* file dari Drive.

# Soal

1. Untuk mempersiapkan pembangunan The Wired, Lain yang berperan sebagai Router membuat tiga Switch/Gateway: Switch 1 menuju dua Entitas yaitu Alice dan Mika, Switch 2 menuju Chisa, sedangkan Switch 3 menuju Knights dan Eiri. Kelima Entitas tersebut dikonfigurasi sebagai Client di GNS3. [GUNAKAN PREFIX IP MASING-MASING KELOMPOK]

Konfigurasi topologi jaringan yang kami buat (sedikit gabung dengan soal 2):

![](img/2026-09-17-12-41-55-image.png)

2. Karena menurut Lain pada saat itu The Wired masih terisolasi dari dunia luar, konfigurasikan router Lain agar dapat tersambung langsung ke jaringan internet publik melalui NAT/DHCP pada interface eth0.

Agar router dapat terhubung ke internet, sambung ke NAT pada eth0.

Kemudian, atur agar router mendapatkan IP secara otomatis dari NAT (eth0) melalui DHCP.

![](img/2026-09-17-12-51-34-image.png)

Cek melalui console -> ip a

![](img/2026-09-17-12-56-13-image.png)

3. Setelah router Lain terhubung ke internet, pastikan seluruh Entitas (Client) di bawah Switch 1, Switch 2, dan Switch 3 dapat saling terhubung dan berkomunikasi satu sama lain melalui konfigurasi routing.

Agar client bisa berkomunikasi satu sama lain, masing-masing dari mereka perlu punya IP. Dan mereka terhubung ke Switch sebagai Gateway, sehingga Switch juga perlu di set IP nya. Setting IP dilakukan secara statis sesuai prefix kelompok.

Set Switch IP pada router:

![](img/2026-09-17-12-58-21-image.png)

Kemudian, masing-masing client perlu IP statis juga, dengan catatan:

* Switch 1, gateway: **192.234.1.1**

* Switch 2, gateway: **192.234.2.1**

* Switch 3, gateway: **192.234.3.1**

Setiap client memulai IP dari **.2**.

```
# Static config for eth0
auto eth0
iface eth0 inet static
	address 192.234.1.2
	netmask 255.255.255.0
	gateway 192.234.1.1
```

4. Lain ingin agar setiap Entitas (Client) memiliki kemandirian di The Wired. Konfigurasikan firewall/iptables (NAT Masquerade) dan DNS resolver agar setiap Client dapat terhubung ke internet secara mandiri (dapat melakukan ping ke 8.8.8.8 dan membuka domain web [google.com](http://google.com))

Agar masing-masing client bisa akses internet, router perlu mengaktifkan fitur packet forwarding. Fitur ini ada di kernel Linux. Serta, konfigurasi routing (iptables) perlu ditambah dengan mode NAT Masquerade.

```
# DHCP config for eth0
auto eth0
iface eth0 inet dhcp
	hostname lain
	# IP Forwarding dan NAT agar klien bisa dapat akses internet
	up sysctl -w net.ipv4.ip_forward=1
	up iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

Kemudian, setiap client perlu mengatur DNS Nameserver agar bisa mengakses domain. Kalau ini ngga ada, client hanya bisa akses IP address (misal 8.8.8.8, bukan google.com).

```
auto eth0
iface eth0 inet static
	...
	up echo "nameserver 8.8.8.8" > /etc/resolv.conf
```

5. Eiri tetap berupaya menanamkan kekacauan ke dalam jaringan. Untuk mengantisipasi restart tiba-tiba, pastikan seluruh konfigurasi jaringan tidak hilang saat semua node di-restart. Buat script verifikasi di /root/cek_status.sh pada router Lain yang menampilkan ringkasan interface (ip -br a) dan status tabel NAT (iptables -t nat -L -v -n) setelah reboot.

```
#!/bin/bash
# /root/cek_status.sh

echo "Interface: "
ip -br a
echo ""

echo "Status tabel NAT (Masquerade): "
iptables -t nat -L -v -n
echo "============================================="

```

![](img/2026-09-17-13-14-14-image.png)

6. Mika mencurigai adanya anomali traffic pada segmen jaringannya. Jalankan generator traffic berikut (link file) pada node Mika, lalu lakukan packet sniffing menggunakan Wireshark pada interface node Mika. Terapkan display filter khusus untuk menyaring paket yang berprotokol DNS atau ICMP. Tunjukkan screenshot hasil filter beserta ringkasan paket yang lolos.

Script dari soal:

```

```

![](img/2026-09-17-13-19-18-image.png)

Sebelum menjalankan, klik kana pada kabel antara **Mika** dan **Switch**, kemudian **Start Capture**.

Wireshark akan terbuka dan mulai capture packet yang dikeluarkan/diterima Mika.

Untuk filter packet, gunakan: `dns or icmp`

![](img/2026-09-17-13-19-40-image.png)



7. Chisa memutuskan mendirikan FTP Server pada node miliknya dengan shared folder di /var/wired/data. Terapkan kebijakan akses: user alice (hak akses read & write), user mika (dibatasi read-only), dan user eiri (dibatasi tanpa izin akses / blacklist). Buktikan konfigurasi dengan membuat file signal_alice.txt dari user alice, dan buktikan penolakan akses saat user eiri mencoba login.

Karena instalasi server FTP ini cukup panjang dan membutuhkan pengulangan jika container stop (ephemeral), kami membuatnya dalam bentuk script pada Chisa.

```bash
#!/bin/bash
# /root/setup_ftp.sh

set -e

echo "=== Install vsftpd ==="

apt update
apt install -y vsftpd

echo "=== Membuat shared folder ==="

mkdir -p /var/wired/data
chmod 777 /var/wired/data

echo "=== Membuat user FTP ==="

useradd -m alice 2>/dev/null || true
echo "alice:alice" | chpasswd

useradd -m mika 2>/dev/null || true
echo "mika:mika" | chpasswd

useradd -m eiri 2>/dev/null || true
echo "eiri:eiri" | chpasswd

echo "=== Konfigurasi vsftpd ==="

cat > /etc/vsftpd.conf <<'EOF'
listen=YES
listen_ipv6=NO
anonymous_enable=NO
local_enable=YES
write_enable=YES
chroot_local_user=YES
local_root=/var/wired/data
allow_writeable_chroot=YES
userlist_enable=YES
userlist_file=/etc/vsftpd.userlist
userlist_deny=YES
user_config_dir=/etc/vsftpd_user_conf
EOF

echo "=== Membuat userlist ==="

cat > /etc/vsftpd.userlist <<'EOF'
# Blacklist eiri
eiri
EOF

echo "=== Membuat konfigurasi per-user ==="

mkdir -p /etc/vsftpd_user_conf

cat > /etc/vsftpd_user_conf/mika <<'EOF'
write_enable=NO
EOF

cat > /etc/vsftpd_user_conf/alice <<'EOF'
# Alice memiliki akses Read-Write
EOF

echo "=== Restart vsftpd ==="

service vsftpd restart

echo "=== Selesai ==="

```

![](img/2026-09-17-13-27-46-image.png)

![](img/2026-09-17-13-29-09-image.png)

Pembuktian FTP:

* Alice

```
# Buat tanda
$ echo "Alice masuk woyy." > signal_alice.txt

# Login sebagai Alice
$ lftp -u alice,alice 192.234.2.2
lftp alice@192.234.2.2:~> put signal_alice.txt 
18 bytes transferred         
                       
lftp alice@192.234.2.2:/> ls
-rw-------    1 1000     1000           18 Sep 16 09:10 signal_alice.txt

lftp alice@192.234.2.2:/>
```

* Eiri:

```
$ lftp -u eiri,eiri 192.234.2.2

lftp eiri@192.234.2.2:~> ls
ls: Login failed: 530 Permission denied. 
```

8. Kelompok rahasia Knights perlu mengirimkan dokumen laporan intelijen ke FTP Server Chisa. Lakukan koneksi FTP client dari node Knights ke FTP Server Chisa menggunakan akun alice. Upload file berikut (link file). Analisis sesi Wireshark dan sebutkan: perintah FTP untuk upload (STOR), kode status sukses server (226), dan port data TCP yang dinegosiasikan pada mode PASV.

Di Knights, unduh file dan upload ke FTP:

```
$ ./gdrive_download.sh 1lFepK4wFmx55PnRki3NsHW-ivudSR0vg knights_report.zip
./gdrive_download.sh: line 45: warning: command substitution: ignored null byte in input
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   772  100   772    0     0    549      0  0:00:01  0:00:01 --:--:--   550
Downloaded to knights_report.zip

$ lftp -u alice,alice 192.234.2.2
lftp alice@192.234.2.2:~> put knights_report.zip 
772 bytes transferred

```

Analisa Wireshark dengan membuka koneksi Wireshark dari Knights, dan filter `ftp or ftp-data`

![](img/2026-09-17-13-31-49-image.png)

* perintah FTP untuk upload (STOR) & kode status sukses server (226)

![](img/2026-09-17-13-32-20-image.png)

* port data TCP yang dinegosiasikan pada mode PASV.

![](img/2026-09-17-13-32-37-image.png)

9. Mika mengakses dokumen Protokol Tujuh di (link file) dari FTP Server Chisa. Dari node Mika, unduh file tersebut menggunakan akun mika. Setelah itu, buktikan pembatasan read-only dengan mencoba mengunggah file baru dari akun mika, dan tunjukkan pesan error respon server (error 550 Permission denied) saat mika mencoba melakukan upload.

Chisa:

```
$ cd /var/wired/data/
$ ./gdrive_download.sh 1tKZu0rcti4t-fXX4jtXDSKDBWzsawfoN protocol7_manifesto.zip
./gdrive_download.sh: line 45: warning: command substitution: ignored null byte in input
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1044  100  1044    0     0    762      0  0:00:01  0:00:01 --:--:--   762
Downloaded to protocol7_manifesto.zip

```

Mika:

```
$ lftp -u mika,mika 192.234.2.2
lftp mika@192.234.2.2:~> ls
-rw-r--r--    1 0        0            1044 Sep 16 19:41 protocol7_manifesto.zip
lftp mika@192.234.2.2:/> get protocol7_manifesto.zip 
1044 bytes transferred
lftp mika@192.234.2.2:/> quit

$ ls
protocol7_manifesto.zip  traffic_protocol7.sh

```

Mika coba upload:

```
$ lftp -u mika,mika 192.234.2.2
lftp mika@192.234.2.2:~> put protocol7_manifesto.zip 
put: Access failed: 550 Permission denied. (protocol7_manifesto.zip)
lftp mika@192.234.2.2:/> 

```

10. Knights melancarkan uji ketahanan koneksi ke server Chisa untuk menguji latensi jaringan The Wired. Kirimkan paket ping dari node Knights ke node Chisa dengan payload khusus 128 bytes dan interval 0.3 detik sebanyak 77 paket (ping -c 77 -s 128 -i 0.3 <IP_Chisa>). Buka Wireshark, catat nilai ICMP Type dan Code untuk Echo Request vs Echo Reply, serta analisis packet loss dan RTT (min/avg/max).

![](img/2026-09-17-13-39-03-image.png)

Request:

![](img/2026-09-17-13-39-24-image.png)

Reply:

![](img/2026-09-17-13-39-40-image.png)

11. Buktikan kelemahan protokol Telnet dengan membuat akun phantom_user dan password wired_ghost pada layanan telnetd di node Chisa. Lakukan login Telnet dari node Eiri ke node Chisa dan tangkap sesi menggunakan Wireshark. Tunjukkan kredensial plain text melalui fitur Follow TCP Stream, serta jelaskan mengapa setiap karakter terkirim dalam paket TCP terpisah.

Tes telnetd:

```
root@eiri:~# telnet 192.234.2.2
Trying 192.234.2.2...
Connected to 192.234.2.2.
Escape character is '^]'.
Linux 6.8.0-90-generic (chisa) (pts/1)

chisa login: 
Password: 
Login incorrect

chisa login: phantom_user
Password: 
8888888b.           888      d8b 888b    888          888
888  "Y88b          888      Y8P 8888b   888          888
888    888          888          88888b  888          888
888    888  .d88b.  88888b.  888 888Y88b 888  .d88b.  888888
888    888 d8P  Y8b 888 "88b 888 888 Y88b888 d8P  Y8b 888
888    888 88888888 888  888 888 888  Y88888 88888888 888
888  .d88P Y8b.     888 d88P 888 888   Y8888 Y8b.     Y88b.
8888888P"   "Y8888  88888P"  888 888    Y888  "Y8888   "Y888

  DebiNet - Lightweight Debian-based Networking Toolbox
  Type "debinet-tools" for available utilities
$ uname -a
Linux chisa 6.8.0-90-generic #91-Ubuntu SMP PREEMPT_DYNAMIC Tue Nov 18 14:14:30 UTC 2025 x86_64 GNU/Linux

```

Wreshark -> Trace pacet -> Ambl pacet telnet -> Follow TCP Stream

![](img/2026-09-17-13-42-07-image.png)

Kredensial terlihat jelas di capture Wireshark.

Telnet secara default beroperasi pada mode "Character-at-a-time" (Karakter-demi-karakter). Setiap kali menekan satu tombol di keyboard (misalnya huruf 'p' pada kata 'phantom'), klien Telnet langsung membungkus huruf tersebut ke dalam satu paket TCP dan mengirimkannya ke server. Server menerima huruf 'p' tersebut, memprosesnya, dan mengirimkan kembali (memantulkan/echo) huruf 'p' ke klien dalam paket TCP terpisah agar huruf tersebut muncul di layar monitor klien.

12. Alice mencurigai Knights menjalankan beberapa layanan rahasia di node-nya. Lakukan pemindaian port dari node Alice ke node Knights menggunakan Netcat (nc) untuk memeriksa port 22 (SSH) dan 80 (HTTP) dalam keadaan terbuka, serta port rahasia 7777 dalam keadaan tertutup. Analisis di Wireshark perbedaan TCP Flag yang dikembalikan antara port terbuka (SYN-ACK) dengan port tertutup (RST-ACK).

Buka port 80 & 22

```
root@knights:~# nc -lvp 80 &
[1] 362
root@knights:~# Listening on 0.0.0.0 80

root@knights:~# nc -lvp 22 &
[2] 363
root@knights:~# Listening on 0.0.0.0 22

root@knights:~#
```

Kemudian, cek di Alice:

```
root@alice:~# nc -zv 192.234.3.2 22 80

Connection to 192.234.3.2 22 port [tcp/ssh] succeeded!
Connection to 192.234.3.2 80 port [tcp/http] succeeded!
root@alice:~# nc -zv 192.234.3.2 7777 
nc: connect to 192.234.3.2 port 7777 (tcp) failed: Connection refused
root@alice:~# 
```

Capture Wireshark:

![](img/2026-09-17-13-44-24-image.png)



SYN-ACK (port terbuka):

![](img/2026-09-17-13-44-49-image.png)

RST-ACK (port tertutup):

![](img/2026-09-17-13-45-14-image.png)

13. Lain memerintahkan agar administrasi jarak jauh menggunakan SSH secara aman tanpa password. Install OpenSSH server pada node Knights, buat pasangan kunci SSH (ssh-keygen) pada node Mika untuk user mika_admin, dan konfigurasikan public key authentication (PasswordAuthentication no). Lakukan koneksi SSH dari node Mika ke node Knights, tangkap sesi menggunakan Wireshark, identifikasi paket Protocol Version Exchange dan Key Exchange, serta jelaskan mengapa kredensial tidak terlihat dalam bentuk teks terbuka seperti pada Telnet.

Knights:

* Install OpenSSH Server

```
root@knights:~# apt-get update && apt-get install openssh-server -y
```

* Aktifkan Public Key & Password authentication

```
root@knights:~# sed -i 's/^#\s*PubkeyAuthentication.*/PubkeyAuthentication yes/' /etc/ssh/sshd_config
root@knights:~# sed -i 's/^#\s*PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config
root@knights:~# sed -i 's/^#\?PasswordAuthentication.*/PasswordAuthentication yes/' /etc/ssh/sshd_config

root@knights:~# grep -E "PasswordAuthentication|PermitRootLogin|Pubkey" /etc/ssh/sshd_config
PermitRootLogin yes
PubkeyAuthentication yes
PasswordAuthentication yes

```

* Restart SSH

```
root@knights:~# service ssh restart
```

* Tambah user `mika_admin`

```
root@knights:~# useradd -m mika_admin && echo "mika_admin:mika_admin" | chpasswd
```

Mika:

* Buat SSH private/public key

```
root@mika:~# ssh-keygen -t rsa -b 2048 -N '' -f ~/.ssh/id_rsa Generating public/private rsa key pair.
Created directory '/root/.ssh'.
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:yvXJFXw5C7tRoE8agR9eNVNzOg1kO/ejB3p1ZObbFdk root@mika
The key's randomart image is:
+---[RSA 2048]----+
|         .. .oBoo|
|        . .+.o O=|
|         ooo= XoE|
|          o= * @o|
|        S . =..o=|
|     . o o o.o+ *|
|      o   +..o o.|
|            . .  |
|                 |
+----[SHA256]-----+

```

* Tambahkan SSH key ke Knights

```
root@mika:~# ssh-copy-id -o StrictHostKeyChecking=no -i ~/.ssh/id_rsa.pub mika_admin@192.234.3.2
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
mika_admin@192.234.3.2's password: 

Number of key(s) added: 1

Now try logging into the machine, with: "ssh -i /root/.ssh/id_rsa -o 'StrictHostKeyChecking=no' 'mika_admin@192.234.3.2'"
and check to make sure that only the key(s) you wanted were added.

```

* Login

```
root@mika:~# ssh mika_admin@192.234.3.2Linux knights 6.8.0-90-generic #91-Ubuntu SMP PREEMPT_DYNAMIC Tue Nov 18 14:14:30 UTC 2025 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
8888888b.           888      d8b 888b    888          888
888  "Y88b          888      Y8P 8888b   888          888
888    888          888          88888b  888          888
888    888  .d88b.  88888b.  888 888Y88b 888  .d88b.  888888
888    888 d8P  Y8b 888 "88b 888 888 Y88b888 d8P  Y8b 888
888    888 88888888 888  888 888 888  Y88888 88888888 888
888  .d88P Y8b.     888 d88P 888 888   Y8888 Y8b.     Y88b.
8888888P"   "Y8888  88888P"  888 888    Y888  "Y8888   "Y888

  DebiNet - Lightweight Debian-based Networking Toolbox
  Type "debinet-tools" for available utilities
$ whoami
mika_admin

```
14. Setelah gagal mengakses FTP, Eiri melancarkan serangan brute-force terhadap form login web Alice. Analisis file capture wired_bruteforce.pcapng untuk mengidentifikasi alamat IP penyerang, target IP beserta port yang diserang, password user lain_admin yang berhasil ditembus, serta web server software dan versi yang dilaporkan pada response header. Validasi temuan kalian pada socket server: (link file) nc [IP_Group] 3401 

Connect ke nc dulu

<img width="512" height="336" alt="image" src="https://github.com/user-attachments/assets/8ae606e8-67e7-4d61-ac30-37ad39e00ff0" />

intinya ditanyain yang nyerang siapa yang di serang apa dan portnya berapa, password user lain_admin dan web server software dan versi berapa yang dilaporkan di response header?

1) Nyari tau IP yang nyerang dan yang di serang
Karena ini bruteforce ke form login kita filter aja Post ke login.php
```text
http.request.method == "POST" && http.request.uri == "/login.php"
```

<img width="1920" height="1128" alt="image" src="https://github.com/user-attachments/assets/d7645418-ffaa-47f4-bc49-b207b560b2e0" />

Bisa dilihat disini tuh ip  172.26.7.50 nyerang dan spam ke ip 172.26.7.100
jadi yg nyerang tuh 172.26.7.50  dan yang di serang  172.26.7.100 untuk portnya 8080

2) nyari password user admin_lain sama web servernya pake filter ini

```text
frame contains "lain_admin"
```

<img width="1920" height="1128" alt="image" src="https://github.com/user-attachments/assets/87021b6d-b13f-41ce-9f71-d46455be495f" />

Follow TCP Stream 

<img width="1920" height="1128" alt="image" src="https://github.com/user-attachments/assets/9939b0f9-841b-4ab9-80d1-7a20b345e5ba" />

Ketemu Password sama Softwarenya

FLAG:
```text
KOMJAR26{W1r3d_Brut3_FGiR2LkTjDaBkskZAuazHKpIM}
```

15. Eiri menyusup ke ruang server dan memasang perangkat keyboard USB berbahaya pada node Alice. Buka file capture wired_usb_hid.pcap, identifikasi Vendor ID dan Product ID perangkat USB dari deskriptor USB, alamat nomor device USB, serta pesan rahasia yang berhasil dicuri dari keystroke. Validasi temuan kalian pada socket server: (link file) nc [IP_Group] 3402 

<img width="1245" height="652" alt="image" src="https://github.com/user-attachments/assets/41e33dc4-6195-49ed-ac1b-093b02821ab7" />

kita ditanya Vendor ID, Product Id USB HID Devicenya, USB device address yg digunakan dan disuruh decode USB HID Keystroke.

1) Cari Vendor ID dan Product ID

```text
usb.idVendor
```
<img width="1920" height="1128" alt="image" src="https://github.com/user-attachments/assets/31cb472c-963e-44ac-ba8e-1b5a2f6524ff" />

Disini id vendornya 0x046d kebetulan nemu jg id productnya 0xc31c tapi klo mau filter sendiri bisa jg pake:
```text
usb.idProduct 
```

2) Nyari adressnya
```text
usb.device_address!=0
```

<img width="1920" height="1128" alt="image" src="https://github.com/user-attachments/assets/02d6b6a1-1ebd-47ab-a3f3-6854eb2f17fc" />

Disini Device Addressnya 7
3) Decode
```text
usb.capdata != 00:00:00:00:00:00:00:00
```
Pake filter itu lalu
<img width="1920" height="1128" alt="image" src="https://github.com/user-attachments/assets/4cc06bdc-f0f0-4ded-af2a-21c9584045e8" />

cek USB URB di Leftover Capture Data, ambil bytes ketiga dari semua itu berikut contoh yg udah diambil
```text
1a 0c 15 08 07 2d 13 15 12 17 12 06 12 0f 2d 24 2d 0c 16 2d 04 0f 0c 19 08 2d 1f 27 1f 23
```
Decode aja ini tabelnya di halaman 90
```text
https://drive.google.com/file/d/1QKb0sZ2LWxbQ_jZrwCXtC1R3McUKuE3E/view 
```
intinya dapet… 

```text
 Wired_Protocol_7_is_alive_2026
```
FLAG:
```text
 KOMJAR26{USB_K3ystr0k3_8nghqGD2krJSejHMYiGM4uxdr}
```
16. Eiri meletakkan file malware di server. Dari file capture wired_ftp_theft.pcap, lakukan analisis lalu lintas FTP untuk mengidentifikasi alamat IP server FTP penyerang, banner software FTP yang digunakan, kredensial login penyerang, serta ukuran (size in bytes) dari file malware knights_payload.exe yang diunduh. Validasi temuan kalian pada socket server: (link file) nc [IP_Group] 3403 

<img width="1156" height="766" alt="image" src="https://github.com/user-attachments/assets/e00326a6-eada-47ef-a1ff-d2b43e5a299f" />

1) Filter biar yg muncul tuh traffic download dari file ftp aja
```text
ftp.request.command == "RETR"
```
<img width="1920" height="1140" alt="image" src="https://github.com/user-attachments/assets/6fd76024-cb86-42f6-a095-def6488dce79" />

Nah ada 3 tapi yg di tanya malware jadi kita tcp stream yg knight_payload.exe oyaa untuk What is the IP address of the FTP server used to download the malware? itu ada di destination.
<img width="1920" height="1140" alt="image" src="https://github.com/user-attachments/assets/af21c131-f8f5-4790-88e5-d39d5442d06b" />

ini untuk tampilan tcp streamnya yang ditanyain tadi What FTP server software banner is returned upon connection? itu ada “vsftpd 3.0.5” terus user password dan size jga udh ada di sini.

FLAG: 
```text
KOMJAR26{FTP_Th3ft_JP7yDIDYOlzCeSL40aNnnWcV9}
```

17. Alice membuat halaman web di node-nya. Eiri memanfaatkan celah untuk mengunduh payload berbahaya ke sistem Alice. Analisis file capture wired_http_c2.pcap untuk mengidentifikasi nama domain (Host) tempat malware diunduh, alamat IP server penyerang, nama file executable malware yang diunduh, serta kode status HTTP yang dikembalikan. Validasi temuan kalian pada socket server: (link file) nc [IP_Group] 3404 

<img width="1053" height="478" alt="image" src="https://github.com/user-attachments/assets/14cacb4c-b778-4cd5-b56a-c9c8ce5d634a" />

1) Menyaring trafic yang request mengunduh atau mengakses file exe
```text
http.request.uri contains ".exe"
```
<img width="1920" height="1128" alt="image" src="https://github.com/user-attachments/assets/9f030759-ac46-4def-a6d4-ec4f44b1e8d7" />
Dari sini langsung HTTP/TCP Stream aja

<img width="1920" height="1128" alt="image" src="https://github.com/user-attachments/assets/2a78c8dc-2580-4e75-8bb0-25d88504ebfb" />

Nah dapet tuh host, nama file malware payload sama http status responses untuk ipnya ada di gambar sebelumnya.

FLAG: 
```text
KOMJAR26{Navi_C2_D0wnl04d_sopLvkEmLS99gMVQOcvru3FWj}
```

18. Eiri mengubah taktik penyerangan dengan menanamkan file malware menggunakan protokol file sharing SMB. Analisis file capture wired_smb_transfer.pcapng untuk mengidentifikasi nama protokol jaringan yang dieksploitasi, IP pengirim dan penerima, folder tujuan penyimpanan malware pada sistem korban, serta nama file executable malware yang ditransfer. Validasi temuan kalian pada socket server: (link file) nc [IP_Group] 3405

<img width="1129" height="604" alt="image" src="https://github.com/user-attachments/assets/b78a63b4-b8a7-4179-9247-160d7f01a566" />

1) Filter supaya hanya menampilkan trafic jaringan yang sedang mentransfer file exe.
```text
smb2.filename contains "exe"
```
<img width="1920" height="1128" alt="image" src="https://github.com/user-attachments/assets/db13c895-5f8d-4d5a-ab4f-a81d7da3e443" />

protocolnya SMB2 untuk ip yg ngirim malware 10.7.3.100 dan ip yg nerima malware 10.7.1.50 lalu kita buka salah satu terus buka SMB2 dan Guid Handle

<img width="1920" height="1128" alt="image" src="https://github.com/user-attachments/assets/8e40584f-57d0-41d0-a118-2adb6d6fa04d" />

bakal ada directory sama nama filenya.
FLAG: 
```text
KOMJAR26{SMB_Tr4nsf3r_CGxOjOH7b8nlH1ZurdYCQmfih}
```
19. Eiri meneror jaringan dengan mengirimkan email pemerasan melalui protokol SMTP tanpa enkripsi. Analisis file capture wired_smtp_threat.pcap pada stream TCP terkait, identifikasi alamat email korban yang ditargetkan, password korban yang diklaim bocor oleh penyerang, jenis malware yang diinfeksikan, batas waktu (dalam hari) yang diberikan, serta MailClientID yang tercantum pada pesan. Validasi temuan kalian pada socket server: (link file) nc [IP_Group] 3406 

<img width="1231" height="736" alt="image" src="https://github.com/user-attachments/assets/6c734661-2235-46be-a5b8-6c52f2b12ed1" />

intinya disuruh nyari email yg jadi target terus password, jenis malware,deadline dan MailClientIDnya

1) Filter penerima email dalam trafik
```text
smtp.req.command == "RCPT"
```
<img width="1920" height="1140" alt="image" src="https://github.com/user-attachments/assets/d0a01feb-18aa-4144-ab8b-fa2e4719b8db" />

disini ada 3 email tapi yang mencurigakan victim@protocol7.co.jp
saat kita tcp stream 

<img width="1920" height="1140" alt="image" src="https://github.com/user-attachments/assets/4a51b645-b1b5-40a4-8983-57092ffce636" />

Ada percakapan kaya ancaman gituu yang isinya tuh jawaban dari soal 19

FLAG : 
```text
KOMJAR26{SMTP_Ext0rt10n_z27nUNA8Uri3niZLGl03HcMmT}
```
20. Untuk rencana pamungkasnya, Eiri menyembunyikan komunikasi malware di balik saluran terenkripsi TLS. Namun Alice telah menyediakan file keylog untuk mendekripsi lalu lintas data tersebut. Analisis file capture wired_tls_decrypt.pcapng bersama keyslogfile.txt untuk mengidentifikasi versi protokol TLS yang dinegosiasikan, nama domain (SNI) yang diakses, alamat IP server HTTPS penyerang, User-Agent yang digunakan, serta HTTP request method dan path yang tersembunyi di dalam sesi dekripsi. Validasi temuan kalian pada socket server: (link file) nc [IP_Group] 3407 

<img width="1234" height="742" alt="image" src="https://github.com/user-attachments/assets/40f6231b-d672-41f6-9675-bf418941f041" />

1) Okee pertama kita harus masukin dulu keyslogfile.txt ke wiresharknya caranya tuh edit > protocols > TLS > masukin filenya ke (Pre)-Master-Secret log filename.
<img width="1054" height="853" alt="image" src="https://github.com/user-attachments/assets/38b5239b-9b8a-4a59-9383-bfb3189e034d" />

ini tampilan setelahnya

<img width="1920" height="1128" alt="image" src="https://github.com/user-attachments/assets/b77cb066-3ef6-4645-8247-f4b7bcefdbf3" />

Oke untuk pertanyaan pertama What specific TLS protocol version was negotiated for the encrypted communication? ini tuh ada di Transport Layer Security yaitu TLS 1.2 sebenernya kalo jawab TLSv1.2 jugaa benarr.

Oke untuk pertanyaan kedua What domain name (SNI / Host) was requested by the client during the TLS handshake?

<img width="1920" height="1128" alt="image" src="https://github.com/user-attachments/assets/98512780-2646-4a5a-a6a9-eb64d63255c7" />
Aku tls stream yang no 1 atau bisa liat aja itu SNI=example.com 

Pertanyaan ketiga What is the IP address of the HTTPS server? ada disini
<img width="1920" height="1128" alt="image" src="https://github.com/user-attachments/assets/b6bd7ecf-12d0-4c13-a0ad-ad8241bb2dab" />

Pertanyaan selanjutnya What User-Agent string was used by the client during the decrypted HTTP session? filter supaya cuma menampilkan http.request
```text
http.request
```

<img width="1920" height="1128" alt="image" src="https://github.com/user-attachments/assets/5d420f7f-8d43-450b-a8a4-a52d1c2b6194" />

Lalu HTTP stream
<img width="1920" height="1128" alt="image" src="https://github.com/user-attachments/assets/8570d57a-eda2-4901-8db7-d10292db499a" />

Yeyy ketemu 
oke pertanyaan terakhir What HTTP request method and path was sent in the decrypted request? ada disini

<img width="1920" height="1128" alt="image" src="https://github.com/user-attachments/assets/6b376ecf-0c8e-47af-974e-019edafe3fd4" />
sebenernya di info dan http stream tadi jugaa ada.

FLAG:
```text
KOMJAR26{TLS_D3crypt_wE38nr4F0iCb2wBr5wnvLIhqI}
```
