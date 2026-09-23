---
title: "Deep Dive Arsitektur Cisco Software-Defined Access (SDA): LISP, VXLAN, dan ISE"
date: 2026-09-23T20:12:38+07:00
draft: false
description: "Analisis teknis mendalam arsitektur Cisco Software-Defined Access (SDA), membedah pemisahan Control Plane LISP, Data Plane VXLAN, dan Policy Plane Cisco ISE SGT di lingkungan perbankan dan enterprise."
tags: ["networking", "Cisco", "SDA", "LISP", "VXLAN", "Cisco ISE", "Catalyst Center", "Enterprise Networking", "NetDevOps"]
categories: ["catatan", "Enterprise Networking", "SDN Architecture"]
author: "Bahrani"
showToc: true
TocOpen: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
---

## 1. Pendahuluan: Mengapa Model VLAN Tradisional Mulai Ditinggalkan?

Pada arsitektur jaringan kampus klasik berbasis model hierarki 3-Tier (*Core, Distribution, Access*), seluruh segmentasi dan mobilitas bergantung pada Layer 2 VLAN dan Spanning Tree Protocol (STP). Di lingkungan berskala ribuan endpoint (seperti kantor pusat perbankan atau kampus bertingkat), pendekatan ini memunculkan sejumlah batasan fundamental:

1. **VLAN Sprawl & Trunking Bloat:** Setiap kali ada segmentasi baru, VLAN harus di-*trunk* melewati puluhan switch distribution dan core, memperluas ukuran *broadcast domain*.
2. **Kerapuhan Spanning Tree (STP Loop Risk):** Satu kabel *loop* atau switch unmanaged yang terpasang tanpa filter BPDU Guard dapat memicu *broadcast storm* yang melumpuhkan seluruh gedung.
3. **Keterikatan Lokasi (Location-Bound IP):** Ketika pengguna berpindah lantai atau gedung, mereka harus mendapatkan IP subnet baru, atau network engineer harus merelokasi port VLAN secara manual.
4. **Skalabilitas ACL Manual yang Buruk:** Pengaturan keamanan antar-departemen mengandalkan ribuan baris Access Control List (ACL) berbasis IP subnet pada Core Switch/Firewall yang rawan *human error* dan sulit diaudit.

**Cisco Software-Defined Access (SDA)** menyelesaikan tantangan ini dengan memisahkan **Underlay (Jaringan Fisik)** dari **Overlay (Jaringan Logis & Kebijakan Keamanan)**.

---

## 2. Arsitektur Fabric Cisco SDA: Tiga Pilar Protokol Utama

SDA mentransformasikan jaringan access switch menjadi sebuah **Fabric**. Di dalam fabric ini, terdapat 3 protokol utama yang membagi tugas secara modular:

![Arsitektur Fabric Cisco SDA](/img/sda/1.png)

<!-- ==========================================
     PROMPT DIAGRAM 1: ARSITEKTUR 3 PILAR SDA
     ==========================================
     Gunakan prompt di bawah ini pada AI Image Generator / Excalidraw / Eraser.io / draw.io:
     
     PROMPT:
     "A professional enterprise network architecture diagram showing Cisco Software-Defined Access (SDA) three-pillar framework. 
      Top layer: Cisco Catalyst Center (DNA Center) managing orchestration and telemetry. 
      Middle layer: Two distinct boxes side-by-side:
        - Left: 'LISP (Control Plane Node / Map Server)' showing EID-to-RLOC mapping database.
        - Right: 'Cisco ISE (Policy Plane)' showing Dot1X/MAB authentication, pxGrid integration, and SGT Matrix.
      Bottom layer: 'VXLAN (Data Plane Fabric)' showing multiple Fabric Edge switches and Fabric Border switches interconnected via Layer 3 Routed Underlay with bidirectional UDP 4789 encapsulation tunnels. 
      Clean dark-mode or tech blueprint style, modern vector iconography, distinct cyan and deep blue accents, clear typography."
     ========================================== -->

```text
┌────────────────────────────────────────────────────────────────────────┐
│                        CISCO CATALYST CENTER                           │
│        (Automation, Policy Intent, Inventory & Telemetry Assurance)    │
└───────────────────┬────────────────────────────────┬───────────────────┘
                    │                                │
┌───────────────────▼────────────────┐ ┌─────────────▼──────────────────┐
│      LISP (Control Plane)          │ │     CISCO ISE (Policy Plane)    │
│   Database Pemetaan: EID ➔ RLOC    │ │   Dot1X / MAB / SGT / SGACL     │
└───────────────────┬────────────────┘ └─────────────┬──────────────────┘
                    │                                │
                    └────────────────┬───────────────┘
                                     │
                    ┌────────────────▼────────────────┐
                    │       VXLAN (Data Plane)        │
                    │ Enkapsulasi L2 Frame ke UDP L3  │
                    └─────────────────────────────────┘
```

---

### A. Control Plane: LISP (Locator/ID Separation Protocol)

Pada routing IP standar, sebuah alamat IP memiliki dua peran ganda yang menyatu:
* **Identitas (Identity):** *Siapa* host tersebut.
* **Lokasi (Location):** *Di mana* host tersebut berada dalam topologi routing.

LISP memecah peran ganda ini menjadi dua konsep independen:
1. **Endpoint Identifier (EID):** Alamat IP host pengguna (tetap identik ke mana pun host berpindah).
2. **Routing Locator (RLOC):** Alamat IP switch *Fabric Edge* di mana host tersebut saat ini terhubung secara fisik.
![Control Plane](/img/sda/2.png)
<!-- ==========================================
     PROMPT DIAGRAM 2: LISP EID VS RLOC LOOKUP
     ==========================================
     PROMPT:
     "A clean network diagram illustrating LISP (Locator/ID Separation Protocol) operation.
      On the left: 'Host A (EID: 10.1.1.50)' connected to 'Fabric Edge Switch 1 (RLOC: 192.168.100.1)'.
      In the center-top: 'LISP Map Server / Control Plane Node' acting as a phonebook database.
      Arrow 1: Fabric Edge 1 sends query 'Where is Host B (EID: 10.1.2.80)?' to Map Server.
      Arrow 2: Map Server replies 'Host B is at Fabric Edge 2 (RLOC: 192.168.100.2)'.
      On the right: 'Fabric Edge Switch 2' delivering traffic to 'Host B'.
      Minimalist modern tech style, isometric or 2D vector flat design, clear arrows with step numbers."
     ========================================== -->

```text
[ Host A (EID: 10.1.1.50) ] ────► [ Fabric Edge 1 (RLOC: 192.168.100.1) ]
                                              │
                                              ▼ (Query LISP Map Server)
                               "Di mana Host B (EID: 10.1.2.80) berada?"
                                              │
                                              ▼ (Map Server Menjawab)
                               "Host B ada di Fabric Edge 2 (RLOC: 192.168.100.2)"
```

**Keuntungan LISP:**
* **Bebas Broadcast ARP Flooding:** Fabric Edge tidak perlu melakukan *flooding* ARP ke seluruh jaringan; lookup lokasi dilakukan langsung ke LISP Map Server.
* **Seamless Host Roaming:** Laptop yang berpindah access switch tetap mempertahankan alamat IP yang sama tanpa pemutusan sesi TCP.

---

### B. Data Plane: VXLAN (Virtual Extensible LAN)

VXLAN bertugas membawa frame Layer 2 melewati jaringan Underlay Layer 3 murni (routed underlay dengan IS-IS atau OSPF) menggunakan enkapsulasi UDP pada port **4789**.
![Data Plane](/img/sda/3.png)
<!-- ==========================================
     PROMPT DIAGRAM 3: FORMAT HEADER PAKET VXLAN DI SDA
     ==========================================
     PROMPT:
     "Technical packet structure diagram showing Cisco SDA VXLAN packet encapsulation header breakdown.
      Horizontally stacked protocol data units:
      1. 'Outer IP Header' (Source RLOC ➔ Dest RLOC, 20 bytes)
      2. 'UDP Header' (Dest Port 4789, 8 bytes)
      3. 'VXLAN-GPO Header' (8 bytes) highlighting:
         - Flags (8 bits)
         - Group Policy / SGT Field (16 bits)
         - VNI / Virtual Network ID (24 bits)
      4. 'Inner Ethernet Header' (Original MAC Source ➔ MAC Dest, 14 bytes)
      5. 'Original IP Payload' (User Application Data)
      Professional engineering color blocks, clearly labeled bit lengths, crisp contrast."
     ========================================== -->

```text
┌──────────────┬──────────────┬──────────────┬──────────────┬───────────────────┐
│ Outer IP Hdr │ UDP (4789)   │ VXLAN Header │ Inner L2 Hdr │ Original Payload  │
│ (RLOC Source │              │ (VNI + SGT)  │ (MAC Source  │ (IP Packet Data)  │
│  ➔ RLOC Dest)│              │              │  ➔ MAC Dest) │                   │
└──────────────┴──────────────┴──────────────┴──────────────┴───────────────────┘
```

**Poin Krusial VXLAN di SDA:**
* **Virtual Network Identifier (VNI):** Memetakan Virtual Network (VRF) untuk isolasi trafik skala masif.
* **Group-Based Policy Extension (SGT Field):** Header VXLAN Cisco membawa tag identitas numerik (**Security Group Tag / SGT**) langsung di dalam header enkapsulasi paket.

---

### C. Policy Plane: Cisco ISE & Security Group Tag (SGT)

Keamanan di Cisco SDA menerapkan prinsip *Zero-Trust Microsegmentation*:

1. **Authentication (802.1X / MAB / FlexAuth):** Saat perangkat terhubung, Fabric Edge berkomunikasi dengan Cisco ISE via RADIUS.
2. **SGT Assignment:** Cisco ISE memberikan Security Group Tag (16-bit integer) berdasarkan profil pengguna (misal: `SGT 4 = Employees`, `SGT 5 = Contractors`, `SGT 10 = Banking Core Server`).
3. **Matrix SGACL:** Penegakan izin (*Permit/Deny*) ditentukan melalui matriks SGACL yang dipusatkan di Catalyst Center dan di-push ke hardware ASIC switch (*Egress Enforcement*).

```text
               DESTINATION SGT
               ┌───────────────┬────────────────┬──────────────┐
               │ Employees (4) │ Contractors (5)│ Core DB (10) │
┌──────────────┼───────────────┼────────────────┼──────────────┤
│ Employees(4) │    PERMIT     │     DENY       │ PERMIT (443) │
├──────────────┼───────────────┼────────────────┼──────────────┤
│Contractor(5) │     DENY      │    PERMIT      │     DENY     │
└──────────────┴───────────────┴────────────────┴──────────────┘
```

---

## 3. Alur Komunikasi Paket End-to-End (Packet Walkthrough)
![Alur Komunikasi Paket](/img/sda/4.png)
<!-- ==========================================
     PROMPT DIAGRAM 4: END-TO-END PACKET WALKTHROUGH
     ==========================================
     PROMPT:
     "A detailed 5-step network packet walkthrough diagram for Cisco SDA.
      Step 1: Laptop (Host A) authenticates with Cisco ISE via 802.1X at Fabric Edge 1, receiving SGT 4.
      Step 2: Fabric Edge 1 registers Host A IP in LISP Map Server.
      Step 3: Host A sends packet to Server B; Fabric Edge 1 encapsulates with VXLAN header (Outer IP + SGT 4).
      Step 4: Packet traverses L3 Routed Underlay spine switches.
      Step 5: Fabric Edge 2 de-encapsulates, evaluates Egress SGACL matrix (SGT 4 to SGT 10 = Permit 443), and forwards packet to Server B.
      Numbered badges (1 to 5), flowing arrows, network topology layout with switch icons."
     ========================================== -->

Berikut urutan transmisi data dari Host A ke Server B:

1. **Onboarding & Autentikasi:** Host A mencolokkan kabel LAN ke Fabric Edge 1. Switch memicu autentikasi 802.1X ke Cisco ISE. ISE memvalidasi kredensial dan menetapkan `SGT = 4` serta `Virtual Network = CORP_VN`.
2. **Registrasi LISP:** Fabric Edge 1 mendaftarkan IP Host A (EID) ke Control Plane Node (Map Server).
3. **Lookup & Enkapsulasi:** Host A mengirim paket ke Server B. Fabric Edge 1 melakukan query LISP ke Map Server untuk mengetahui RLOC Server B. Map Server mengembalikan IP Fabric Edge 2.
4. **Transmisi VXLAN:** Fabric Edge 1 mengenkapsulasi paket ke dalam frame VXLAN dengan menyematkan `SGT = 4` pada header, lalu mengirimkannya melewati jaringan underlay routed IP.
5. **Egress Enforcement di Edge Tujuan:** Fabric Edge 2 menerima paket VXLAN, membuka enkapsulasi (*de-encapsulation*), membaca `Source SGT = 4` dan `Destination SGT = 10`, lalu mencocokkannya dengan tabel SGACL lokal. Jika diizinkan, paket diteruskan ke port Server B.

---

## 4. Perintah Verifikasi & Troubleshooting CLI Penting

Untuk memvalidasi operasional fabric pada switch Catalyst 9000, gunakan perintah kunci berikut:

```cisco
1. Memeriksa sesi registrasi LISP Control Plane
show lisp instance-id 4099 ipv4 server summary
show lisp instance-id 4099 ipv4 map-cache
!
2. Memeriksa status enkapsulasi Data Plane VXLAN
show platform software vxlan fabric-edge
show interfaces nve 1
!
3. Memeriksa autentikasi endpoint dan alokasi SGT dari Cisco ISE
show authentication sessions
show authentication sessions interface GigabitEthernet1/0/1 details
!
4. Memeriksa policy enforcement SGT / SGACL
show cts role-based permissions
show cts role-based counters
!
5. Memeriksa status CTS/SGT pada interface
show cts interface GigabitEthernet1/0/1
```

---

## 5. Kesimpulan & Rekomendasi Implementasi

Implementasi Cisco SDA memberikan lompatan efisiensi dan keamanan yang signifikan:
* **Zero VLAN Sprawl:** Seluruh kampus cukup menggunakan satu subnet pool dinamis per Virtual Network.
* **Consistent Policy:** Keamanan melekat pada identitas pengguna (SGT), bukan lokasi fisik port switch.
* **Kunci Keberhasilan:** Pastikan jaringan **Underlay** dirancang dengan MTU Jumbo (9100 bytes) dan routing protocol yang deterministik (IS-IS / BFD) sebelum mengaktifkan fabric overlay.

