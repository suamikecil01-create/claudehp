# CLAUDE.md

## CEK IKLAN — STANDING ORDER

Begitu user bilang **"cek iklan"**, langsung kirim laporan di bawah ini tanpa nanya
balik, tanpa minta konfirmasi format, tanpa nanya akun mana.

- Data pakai `date_preset=today`.
- Satu akun saja: **SVO 01** (`2385832538283520`).
- **SVO 03 (`428182959794476`) tidak dipakai.** Jangan tarik datanya, jangan bikin
  bloknya, jangan sebut di laporan.
- Tabel adset tidak dipotong.
- **Adset yang statusnya sudah PAUSED tidak dimunculkan lagi di tabel/blok manapun**
  (tabel 1, blok 0 kontak, blok purchase, kill candidate) — biar enak dibaca. Satu-satunya
  pengecualian adalah blok 3 TOTAL, yang tetap angka level `ad_account` (termasuk spend
  adset yang sudah dipause). Tidak ada lagi status "10 adset 24 jam" yang dikunci/dikecualikan
  — semua adset ACTIVE dinilai dengan aturan yang sama, termasuk bisa jadi kill candidate.

### Urutan output (persis begini, semua blok pakai emot)

**1. 📊 Tabel "ada kontak"**
Heading: `📊 SVO 01 — ada kontak`.
Kolom: Adset, Spend, Kontak, CPL, Pembelian, CPR. Urut spend terbesar.
Tanpa kolom ATC atau kolom tambahan apa pun. Pembelian/CPR kosong ditulis `—`.
Adset 0 kontak **tidak boleh nyempil** di tabel ini, sekalipun cuma satu baris.
Cek ulang sebelum kirim.

**2. ⚫ Blok "0 kontak"**
Heading: `⚫ SVO 01 — 0 kontak`.
Satu baris, dipisah ` · `, format `EMOT NAMA spend`. Nama adset + spend saja.

**3. 💰 TOTAL**
`💰 TOTAL SVO 01`: Spend Rp x · n pembelian · CPR Rp x.
**Wajib diambil dari level `ad_account`, bukan dijumlah dari tabel adset.**
Adset yang dipause siang tetap sudah belanja; kalau dibuang, angka jadi kelihatan
lebih bagus dari kenyataan.

**4. 🛒 Adset yang menghasilkan purchase**
Satu baris: `NAMA (n × CPR) · ...`. Hanya adset yang masih ACTIVE (adset yang sudah
PAUSED tidak dimunculkan lagi, sesuai aturan di atas).
Kalau nol, tulis `Belum ada purchase hari ini`.

**5. 📦 Ringkasan**
- Anggaran direncanakan = jumlah `daily_budget` semua adset yang sempat jalan hari
  ini, ACTIVE maupun PAUSED.
- Sudah jalan dari pagi = total spend.
- Sisa anggaran. Kalau spend melebihi budget tulis `Lewat Rp x`
  (Meta boleh overspend harian sampai 25%).
- Plus total purchase.

**6. 🚫 Kill candidate — dua aturan saja**
- (a) spend ≥ Rp75.000 dan **0 kontak**
- (b) spend ≥ Rp150.000 dan **1 kontak**

Tidak ada pembeda siang/sore. Tidak ada kriteria CPL. Semua yang kena (a) atau (b)
wajib masuk, jam berapa pun laporannya. Jangan bikin kriteria sendiri.

Dikecualikan cuma satu:
1. Adset yang punya purchase hari itu — sebut alasannya singkat, tulis terpisah:
   `X dikecualikan — alasan`.

(Adset yang sudah PAUSED otomatis tidak muncul di blok 1/2, jadi otomatis juga tidak
muncul di sini — tidak perlu dicek lagi.)

Format: bullet nama + alasan singkat; adset 0 kontak yang spend udah lewat batas
boleh digabung satu bullet.
Sebelum kirim, **scan ulang blok 1 dan 2**: tiap baris 0 kontak dengan spend ≥ 75rb,
dan tiap baris 1 kontak dengan spend ≥ 150rb, harus ada di blok ini.
Kalau tidak ada sama sekali, tulis `Belum ada kill candidate`.

### Emot status (wajib di depan nama adset di SEMUA baris, semua blok)

| Emot | Arti |
| --- | --- |
| 🟢 | aman — udah ada purchase hari ini |
| 🟡 | pantau — ada kontak, belum purchase |
| 🔴 | kill candidate — kena aturan (a) atau (b) |
| ⚫ | 0 kontak — spend belum lewat batas 75rb |

Di blok 0 kontak: adset spend < 75rb pakai ⚫, spend ≥ 75rb pakai 🔴 (dan otomatis
muncul di blok kill candidate).

### Aturan teknis

- **Kontak dan CPL** diambil dari
  `cost_per_action_type:offsite_contact_website_add_meta_leads`,
  BUKAN `onsite_conversion_lead_grouped` (sering null padahal kontak ada).
  - Kontak = spend dibagi angka itu, dibulatkan.
  - CPL = angka itu langsung.
  - Kalau kosong, fallback ke
    `cost_per_action_type:onsite_conversion.messaging_conversation_started_7d`.
  - Catatan API: minta field `cost_per_action_type` polos (bukan
    `cost_per_action_type:<sub>`), lalu baca sub-key-nya dari hasil.
- **Pembelian** dari `omni_purchase`, **CPR** dari `cost_per_omni_purchase`.
- Jangan pakai blockquote atau border di output yang mau dicopy.
- **Jangan pause atau ubah status apa pun tanpa perintah user.**
- Selisih purchase level adset vs level akun itu wajar karena atribusi Meta.
  Tetap pakai angka level akun buat TOTAL. Boleh tambah "catatan jujur" singkat
  bernomor di penutup kalau relevan.
- Kalau user kasih perintah matiin pakai angka tanpa nyebut metriknya
  (misal "matiin yang di atas 75k"), **jangan langsung eksekusi** — tanya dulu
  spend atau CPL.
