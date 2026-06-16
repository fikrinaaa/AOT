<div align="center">

<img src="https://img.shields.io/badge/Wazuh-4.14.0-blue?style=for-the-badge&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PHBhdGggZmlsbD0id2hpdGUiIGQ9Ik0xMiAyTDIgN2wxMCA1IDEwLTV6TTIgMTdsOSA1IDktNXYtNmwtOSA1LTktNXoiLz48L3N2Zz4="/>
<img src="https://img.shields.io/badge/DFIR--IRIS-v2.4.20-purple?style=for-the-badge"/>
<img src="https://img.shields.io/badge/Shuffle-SOAR-orange?style=for-the-badge"/>
<img src="https://img.shields.io/badge/Ansible-Automated-red?style=for-the-badge&logo=ansible"/>
<img src="https://img.shields.io/badge/Vagrant-VirtualBox-1868F2?style=for-the-badge&logo=vagrant"/>

# ⚙️ AOT
### **A**utomation **O**rchestration **T**oolkit

> *Deploy platform keamanan siber lengkap — Wazuh, DFIR-IRIS, dan Shuffle SOAR —*
> *hanya dengan satu perintah*

---

**[📖 Panduan Lengkap](#-cara-menjalankan)** · **[⚡ Quick Start](#-quick-start)** · **[🏗️ Arsitektur](#-arsitektur)** · **[❓ FAQ](#-faq)**

</div>

---

## ✨ Apa itu AOT?

AOT adalah **otomasi deployment platform SOC (Security Operations Center)**. Alih-alih menginstall dan mengkonfigurasi Wazuh, IRIS, dan Shuffle secara manual, AOT melakukannya secara otomatis dalam satu perintah.

```
vagrant up
```

Dengan itu, tiga VM akan dibuat, Docker diinstall, dan semua tools berjalan otomatis. AOT juga dilengkapi workflow respons insiden otomatis yang menangani alert apa pun selama memiliki indikator IP sumber — mulai dari triase, pemeriksaan reputasi ke VirusTotal, dokumentasi case di IRIS, hingga pemblokiran IP melalui Wazuh Active Response setelah disetujui analis.

### 🧩 Stack Tools

| Tool | Versi | Peran |
|------|-------|-------|
| [**Wazuh**](https://wazuh.com) | v4.14.0 | SIEM — Deteksi & monitoring ancaman real-time |
| [**DFIR-IRIS**](https://dfir-iris.org) | v2.4.20 | Case Management — Pencatatan & pengelolaan insiden |
| [**Shuffle**](https://shuffler.io) | Latest | SOAR — Otomasi workflow respons insiden |
| [**Telegram Bot**](https://core.telegram.org/bots) | — | Notifikasi alert & permintaan approval ke analis |

---

## 🏗️ Arsitektur

```
┌─────────────────────────────────────────────────────────┐
│                            PC                           │
│                                                         │
│  $ vagrant up                                           │
│       │                                                 │
│       ▼                                                 │
│  ┌──────────────────────────────────────────────────┐   │
│  │              VirtualBox (Host-Only Network)      │   │
│  │                  192.168.56.0/24                 │   │
│  │                                                  │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────┐  │   │
│  │  │ aot-wazuh    │  │ aot-iris     │  │aot-    │  │   │
│  │  │192.168.56.10 │  │192.168.56.11 │  │shuffle │  │   │
│  │  │              │  │              │  │.56.12  │  │   │
│  │  │  🔍 Wazuh   │  │  📋 IRIS    │  │⚡Shuf  │  │   │
│  │  │   Server     │  │             │   │  fle   │  │   │
│  │  │  (Docker)    │  │  (Docker)   │   │(Docker)│  │   │
│  │  └──────────────┘  └──────────────┘  └────────┘  │   │
│  │                          ▲                       │   │
│  │                    Ansible                       │   │
│  │                                                  │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### Alur Otomasi

```
vagrant up
    │
    ├─► Buat VM aot-wazuh   → aktifkan SSH → install Docker → deploy Wazuh
    ├─► Buat VM aot-iris    → aktifkan SSH → install Docker → deploy IRIS  
    └─► Buat VM aot-shuffle → aktifkan SSH → install Docker → deploy Shuffle
                                                      │
                                              Ansible berjalan
                                           dari dalam VM ini ke
                                              semua VM lainnya
```

### Alur Workflow Respons Insiden

```
Wazuh Alert (webhook)
    │
    ▼
Detection Workflow ──┬──► Notification Workflow  (notif awal ke Telegram)
                      │
                      └──► Triage and Analysis Workflow
                                  │
                                  ├─ Create Case (IRIS)
                                  ├─ Cek reputasi IP (VirusTotal)
                                  ├─ Tambah IOC ke case (IRIS)
                                  │
                                  ▼
                           Send User Input
                           (analis memilih Lanjutkan / Hentikan)
                                  │
                ┌─────────────────┴─────────────────┐
                ▼                                     ▼
        Lanjutkan                              Hentikan
                │                                     │
                ▼                                     ▼
   Response and Reporting Workflow          Workflow berhenti
   (Wazuh Active Response: firewall-drop    (instruksi tutup case
    + notifikasi ringkasan ke Telegram)      sudah ada di pesan)
```

---

## 💻 Spesifikasi

### VM yang Dibuat

| VM | IP | RAM | CPU | Tool |
|----|----|-----|-----|------|
| `aot-wazuh` | 192.168.56.10 | 8 GB | 4 core | Wazuh Server |
| `aot-iris` | 192.168.56.11 | 4 GB | 2 core | DFIR-IRIS |
| `aot-shuffle` | 192.168.56.12 | 8 GB | 2 core | Shuffle SOAR |

---

## ⚡ Quick Start

> Untuk panduan lengkap dengan penjelasan setiap langkah, lihat [Cara Menjalankan](#-cara-menjalankan) di bawah.

```bash
# 1. Install VirtualBox → https://virtualbox.org
# 2. Install Vagrant   → https://developer.hashicorp.com/vagrant
# 3. Clone repo
git clone https://github.com/fikrinaaa/AOT.git
cd AOT

# 4. Install Ansible collections
ansible-galaxy collection install -r ansible/requirements.yml

# 5. Jalankan!
vagrant up
```

Tunggu 30-60 menit, lalu akses:
- 🔍 **Wazuh** → https://192.168.56.10
- 📋 **IRIS** → https://192.168.56.11
- ⚡ **Shuffle** → https://192.168.56.12:3443

---

## 📖 Cara Menjalankan

### Langkah 1 — Install VirtualBox

VirtualBox adalah software yang menjalankan VM di laptop kamu.

1. Buka https://www.virtualbox.org/wiki/Downloads
2. Pilih installer sesuai OS laptop kamu
3. Jalankan installer (klik Next terus, semua default sudah cukup)
4. Restart laptop jika diminta

> ⚠️ **Pengguna Windows:** pastikan fitur virtualisasi (VT-x / AMD-V) aktif di BIOS. Cara cek: buka Task Manager → tab Performance → CPU → lihat apakah "Virtualization: Enabled".

> ⚠️ **Pengguna Windows dengan Hyper-V aktif:** Hyper-V bisa konflik dengan VirtualBox. Matikan Hyper-V via PowerShell (sebagai Administrator):
> ```
> bcdedit /set hypervisorlaunchtype off
> ```
> Lalu restart laptop.

---

### Langkah 2 — Install Vagrant

Vagrant adalah tool yang mengotomasi pembuatan VM dan menghubungkannya dengan Ansible.

1. Buka https://developer.hashicorp.com/vagrant/downloads
2. Pilih installer sesuai OS
3. Jalankan installer
4. Verifikasi — buka terminal baru dan ketik:

```bash
vagrant --version
# Output: Vagrant 2.x.x
```

---

### Langkah 3 — Install Ansible

> **Prasyarat Khusus Pengguna Windows:** Ansible tidak bisa diinstall langsung di Windows. Gunakan **WSL2** (Windows Subsystem for Linux). Ikuti langkah di bawah.

#### Windows — Setup WSL2

Buka PowerShell sebagai Administrator:

```powershell
wsl --install
```

Restart laptop. Setelah restart, buka **Ubuntu** dari Start Menu dan buat username + password Linux kamu.

Lanjutkan langkah berikut di dalam terminal Ubuntu (WSL):

```bash
# Aktifkan interop WSL agar bisa jalankan Vagrant dari WSL
sudo sh -c 'echo :WSLInterop:M::MZ::/init:PF > /proc/sys/fs/binfmt_misc/register'

# Buat permanen
echo '[interop]
enabled=true
appendWindowsPath=true' | sudo tee /etc/wsl.conf

# Restart WSL — jalankan di PowerShell:
# wsl --shutdown
# Lalu buka kembali Ubuntu WSL
```

#### Semua OS — Install Ansible

```bash
# Linux / WSL (Ubuntu/Debian)
sudo apt update && sudo apt install -y python3-pip
sudo apt install ansible-core -y

# macOS
brew install python3
pip3 install ansible

# Verifikasi
ansible --version
```

---

### Langkah 4 — Clone Repository

```bash
git clone https://github.com/fikrinaaa/AOT.git
cd AOT
```

---

### Langkah 5 — Install Ansible Collections

```bash
ansible-galaxy collection install -r ansible/requirements.yml
```

Collections yang diinstall:
- `community.docker` — untuk mengelola Docker Compose via Ansible
- `community.general` — tools umum (timezone, hostname, dll)

---

### Langkah 6 — Jalankan!

```bash
vagrant up
```

Proses yang terjadi secara otomatis:

```
[1] Download Ubuntu 22.04 box — hanya pertama kali
[2] Buat 3 VM di VirtualBox
[3] Aktifkan SSH password auth di semua VM
[4] Install sshpass di VM Shuffle
[5] Ansible install dari dalam VM Shuffle
[6] Deploy ke semua VM:
    ├── common  : update apt, install paket dasar, disable swap, set sysctl
    ├── docker  : install Docker Engine + Compose Plugin
    ├── wazuh   : clone wazuh-docker, generate sertifikat, docker compose up
    ├── iris    : clone iris-web, konfigurasi .env, docker compose up
    └── shuffle : clone Shuffle, docker swarm init, docker compose up
[7] Tampilkan ringkasan URL akses
```

Jika berhasil, di akhir terminal akan muncul:

```
TASK [Tampilkan ringkasan akses semua tools]
ok: [localhost] => {
    "msg": [
        "==============================================",
        "  AOT — Deployment Selesai!",
        "==============================================",
        "Wazuh Dashboard : https://192.168.56.10",
        "DFIR-IRIS       : https://192.168.56.11",
        "Shuffle UI      : https://192.168.56.12:3443",
        "=============================================="
    ]
}
```

---

## 🔐 Akses Tools

### Wazuh Dashboard

```
URL      : https://192.168.56.10
Username : admin
Password : SecretPassword 
```

### DFIR-IRIS

```
URL      : https://192.168.56.11
Username : administrator
Password : sesuai iris_passwd di ansible/group_vars/all/main.yml
```

### Shuffle SOAR

```
URL      : https://192.168.56.12:3443
Buat akun admin saat akses pertama via browser
```

---

## 📲 Setup Telegram Bot (Notifikasi & Approval)

Shuffle mengirimkan notifikasi alert dan permintaan approval ke grup Telegram tim.

### Buat Bot via BotFather

1. Buka Telegram → cari **@BotFather** (centang biru)
2. Ketik `/newbot`
3. Isi nama bot: `AOT Alert Bot`
4. Isi username bot: `aot_alert_bot` (harus diakhiri `bot`)
5. Salin **Bot Token** yang diberikan

### Dapatkan Chat ID

1. Buat grup Telegram (contoh: `AOT SOC Alerts`)
2. Tambahkan bot kamu ke grup
3. Tambahkan **@userinfobot** ke grup — ia akan mengirim Chat ID grup
4. Chat ID grup biasanya berformat angka negatif: `-1001234567890`

### Konfigurasi di Shuffle

1. Login ke Shuffle → menu **Apps** → cari `Telegram_Bot` → **Authenticate** → masukkan Bot Token
2. Import workflow dari folder `Shuffle Workflow/`
3. Buka **AOT Phase 1 - Notification Workflow** → klik node `Send_Alert_Notification` → isi `chat_id`
4. Buka **AOT Phase 2 - Triage and Analysis Workflow** → klik node `Send_Notification` → isi `chat_id`
5. Buka **AOT Phase 3 - Send User Input** → klik node `Send_Trigger` → isi `chat_id`
6. Buka **AOT Phase 4 - Response and Reporting Workflow** → klik node `Response_Notification` → isi `chat_id`

---

## 🔗 Integrasi Antar Tools

### Wazuh → Shuffle (Webhook + Active Response)

1. Di Shuffle, buka **AOT Core - Detection Workflow** → klik node webhook → salin **Webhook URL**
2. SSH ke VM Wazuh:

```bash
vagrant ssh wazuh
docker exec -it single-node-wazuh.manager-1 bash
nano /var/ossec/etc/ossec.conf
```

3. Tambahkan di dalam `<ossec_config>`:

```xml
<!-- Integrasi Shuffle: meneruskan alert ke webhook -->
<integration>
  <name>shuffle</name>
  <hook_url>https://192.168.56.12:3443/api/v1/hooks/<ID-WEBHOOK></hook_url>
  <level>3</level>
  <alert_format>json</alert_format>
  <options>SSL_VERIFY=false</options>
</integration>

<!-- Definisi command Active Response firewall-drop -->
<command>
  <name>firewall-drop</name>
  <executable>firewall-drop</executable>
  <timeout_allowed>yes</timeout_allowed>
</command>

<!-- Izinkan firewall-drop dieksekusi via API eksternal (Shuffle) -->
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <timeout>no</timeout>
</active-response>
```

> `firewall-drop` adalah skrip Active Response bawaan Wazuh, sudah tersedia di setiap agent tanpa instalasi tambahan. Blok `<active-response>` di atas yang memungkinkan command ini dipicu dari luar (API/Shuffle), bukan hanya dari rule lokal Wazuh.

4. Restart Wazuh Manager:

```bash
/var/ossec/bin/wazuh-control restart
exit && exit
```

### Shuffle → Wazuh (Active Response API)

Node Active Response di Shuffle membutuhkan JWT token Wazuh sebagai autentikasi. Karena token memiliki masa berlaku, lifetime-nya perlu diperpanjang agar tidak sering expired.

1. SSH ke VM Wazuh:

```bash
vagrant ssh wazuh

# Dapatkan token awal
TOKEN=$(curl -k -u 'wazuh-wui:<PASSWORD_API_WAZUH>' \
  "https://192.168.56.10:55000/security/user/authenticate?raw=true")

# Perpanjang lifetime token (praktis tidak expire)
curl -k -X PUT "https://192.168.56.10:55000/security/config" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"auth_token_exp_timeout": 99999999}'

# Generate token baru dengan lifetime panjang
curl -k -u 'wazuh-wui:<PASSWORD_API_WAZUH>' \
  "https://192.168.56.10:55000/security/user/authenticate?raw=true"
```

2. Salin token yang dihasilkan (`eyJ...`)
3. Di Shuffle: menu **Apps** → `Wazuh` → **Authenticate** → isi URL `https://192.168.56.10:55000` dan field `apikey` dengan token tersebut → **Save**
4. Buka **AOT Phase 4 - Response and Reporting Workflow** → pastikan node `Wazuh_1` (Get Login) dan `Wazuh_2` (Active Response) menggunakan authentication Wazuh yang baru dibuat

> Token perlu di-generate ulang apabila VM Wazuh di-restart. Ulangi langkah generate token di atas dan update authentication di Shuffle.

### Shuffle → DFIR-IRIS (API)

1. Di IRIS: klik ikon user → **My profile** → **Generate API Key** → salin key
2. Di Shuffle: menu **Apps** → `IRIS_v2` → **Authenticate** → isi URL `https://192.168.56.11` dan API Key
3. Buka **AOT Phase 2 - Triage and Analysis Workflow** → pastikan node `Create_Case` dan `Add_IOC_IP` terhubung ke authentication IRIS → aktifkan workflow

### Shuffle → VirusTotal (API)

1. Daftar/login di https://www.virustotal.com → profil → **API Key** → salin key
2. Di Shuffle: menu **Apps** → `VirusTotal` → **Authenticate** → masukkan API Key
3. Buka **AOT Phase 2 - Triage and Analysis Workflow** → pastikan node `IP_Analysis` terhubung ke authentication VirusTotal

---

## 🕹️ Perintah Vagrant

```bash
vagrant up              # Buat semua VM + provisioning
vagrant halt            # Matikan semua VM (data tersimpan)
vagrant up              # Hidupkan kembali (tanpa install ulang)
vagrant provision       # Jalankan ulang Ansible tanpa rebuild VM
vagrant ssh wazuh       # Masuk ke VM Wazuh
vagrant ssh iris        # Masuk ke VM IRIS
vagrant ssh shuffle     # Masuk ke VM Shuffle
vagrant status          # Cek status semua VM
vagrant destroy -f      # Hapus semua VM (permanen!)
```

---

## 🗂️ Struktur Direktori

```
aot/
├── Vagrantfile                        ← konfigurasi VM VirtualBox
├── README.md
├── ansible/
│   ├── ansible.cfg
│   ├── playbook.yml                   ← playbook utama
│   ├── requirements.yml               ← Ansible collections
│   ├── group_vars/
│   │   └── all/main.yml               ← semua variabel (edit di sini)
│   ├── inventory/
│   │   └── hosts                      ← IP statis VM
│   └── roles/
│       ├── common/                    ← setup dasar semua VM
│       ├── docker/                    ← install Docker
│       ├── wazuh_server_docker/       ← deploy Wazuh
│       ├── dfir_iris/                 ← deploy DFIR-IRIS
│       └── shuffle/                   ← deploy Shuffle
└── Shuffle Workflow/
    ├── AOT_Core_Detection_Workflow.json
    ├── AOT_Phase1_Notification_Workflow.json       ← Telegram alert awal
    ├── AOT_Phase2_Triage_and_Analysis_Workflow.json
    ├── AOT_Phase3_Send_User_Input.json             ← approval analis
    └── AOT_Phase4_Response_Workflow.json           ← Active Response + laporan
```

---

## ❓ FAQ

<details>
<summary><b>Berapa lama proses vagrant up?</b></summary>

Pertama kali: 30–60 menit (tergantung kecepatan internet, karena download image Docker cukup besar).

Selanjutnya `vagrant halt` → `vagrant up` hanya butuh ~2–3 menit karena VM sudah ada.
</details>

<details>
<summary><b>Apakah bisa diakses dari perangkat lain di jaringan?</b></summary>

Tidak secara default — network host-only hanya bisa diakses dari laptop yang menjalankan Vagrant. Untuk akses dari perangkat lain, ganti ke `public_network` (bridged) di Vagrantfile:

```ruby
wazuh.vm.network "public_network", bridge: "Ethernet"
```
</details>

<details>
<summary><b>VM timeout saat vagrant up</b></summary>

Kemungkinan laptop kehabisan resource saat boot 3 VM bersamaan. Coba boot satu per satu:

```bash
vagrant up wazuh
vagrant up iris
vagrant up shuffle
```

Atau tambahkan di Vagrantfile:

```ruby
config.vm.boot_timeout = 600
```
</details>

<details>
<summary><b>Active Response gagal dengan pesan "AR command was not sent to any agent"</b></summary>

Pesan ini muncul saat tidak ada Wazuh agent yang terdaftar dan aktif di Manager. Cek agent yang terdaftar:

```bash
vagrant ssh wazuh
docker exec -it single-node-wazuh.manager-1 /var/ossec/bin/agent_control -l
```

Pastikan juga IP atau ID agent pada `agents_list` di node Active Response sesuai dengan agent yang benar-benar terdaftar.
</details>

<details>
<summary><b>Kenapa Triage Workflow tidak punya kondisi berdasarkan Rule ID?</b></summary>

Triage Workflow dirancang generik — setiap alert yang masuk dan memiliki field `data.srcip` akan diproses sama: dicek ke VirusTotal, didokumentasikan ke IRIS, lalu diteruskan ke analis untuk keputusan. Pendekatan ini menghindari kebutuhan mendefinisikan skenario serangan satu per satu, dan menjadikan VirusTotal sebagai penentu objektif apakah IP tersebut layak diblokir.
</details>

<details>
<summary><b>Bagaimana cara update tool ke versi terbaru?</b></summary>

Edit versi di `ansible/group_vars/all/main.yml`:

```yaml
wazuh_version: "4.14.0"   # ganti ke versi terbaru
iris_version:  "v2.4.20"  # ganti ke versi terbaru
```

Lalu jalankan:

```bash
vagrant provision
```
</details>

---

## 📚 Referensi

- [Wazuh Documentation](https://documentation.wazuh.com/current/)
- [DFIR-IRIS Documentation](https://docs.dfir-iris.org/)
- [Shuffle Documentation](https://shuffler.io/docs)
- [Telegram Bot API](https://core.telegram.org/bots/api)
- [Vagrant Documentation](https://developer.hashicorp.com/vagrant/docs)
- [Ansible Documentation](https://docs.ansible.com/)

---

<div align="center">
Made with ❤️
</div>
