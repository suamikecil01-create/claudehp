# CLAUDE.md

## CEK IKLAN — STANDING ORDER

Bantu user cek iklan Meta (tanpa Adcuan). Bahasa Indonesia santai aku-kamu.
**Semua hasil WAJIB tabel.** Jangan tutup laporan pakai pertanyaan.
Begitu user bilang **"cek iklan"**, langsung tarik data dan kirim laporan di bawah
ini tanpa nanya balik.

### Akun

| Akun | ID | Catatan |
| --- | --- | --- |
| SVO 04 Erojan | `1427793538132297` | tampilkan **PALING ATAS** |
| SVO 01 (utama) | `2385832538283520` | |

SVO 03 (`428182959794476`) tidak dipakai.

### Cara tarik data (tiap "cek iklan")

1. SVO 01 level `adset`, `date_preset=today`, filter `amount_spent > 0`, fields:
   `id, name, campaign_name, effective_status, daily_budget, start_time,
   amount_spent, cost_per_action_type, link_click, omni_add_to_cart, omni_purchase`.
   limit 300.
2. SVO 01 level `ad_account`, today: `amount_spent, cost_per_action_type,
   omni_purchase, cost_per_omni_purchase`.
3. SVO 04 level `adset`, today: `name, effective_status, amount_spent, impressions,
   reach, link_click, omni_purchase` (+ `cost_per_action_type` buat hitung kontak).

Catatan API: minta field `cost_per_action_type` polos, lalu baca sub-key-nya dari hasil.

### Hitung kontak

- Kontak web = spend ÷ `cost_per_action_type:offsite_contact_website_add_meta_leads` (bulatkan).
- WA langsung = spend ÷ `cost_per_action_type:onsite_conversion.messaging_conversation_started_7d` (bulatkan).
- Kontak total = web + WA. Kosong = 0.
- CPL = spend ÷ kontak total. Purchase dari `omni_purchase`, CPR dari `cost_per_omni_purchase`.
- "Masih nyala" = `effective_status` ACTIVE saja. Adset PAUSED **tidak** ditampilkan
  di tabel utama (kecuali ditandai di tabel purchase).
- Adset tanpa `daily_budget` = CBO, tulis `nama (CBO nama campaign)`.

### Format laporan (urut persis)

1. **SVO 04 Erojan**: | Adset | Spend | Tayangan | Jangkauan | Klik | Kontak | Purchase |
2. **SVO 01 ada kontak** (tulis jam cek): | Adset | Spend | Kontak web | WA | Total | CPL | — urut spend terbesar.
3. **SVO 01 0 kontak**: | Adset | Spend | Klik tautan | ATC |
4. **CPL di atas 100rb**: | Adset | Spend | Kontak | CPL | Catatan |
5. **TOTAL SVO 01** (angka level akun): | Spend | Kontak | CPL | Purchase | CPR |
   Di bawahnya sebut sekali: kontak cuma dari Meta (tanpa Adcuan), jadi bisa sedikit
   lebih kecil dari Adcuan.
6. **Adset yang menghasilkan purchase**: | Adset | Spend | Purchase | CPR | — yang
   paused ditandai "sudah di-pause".
7. **Ringkasan**: | | Anggaran | Duit keluar | % | Sisa | Purchase |
   - "Total duit keluar hari ini": anggaran = `daily_budget` semua ABO yang spend>0
     (termasuk paused) + Rp1jt per campaign CBO yang nyala; duit keluar = spend akun.
   - "Dari yang masih nyala": anggaran = `daily_budget` ABO ACTIVE + Rp1jt per CBO
     ACTIVE; duit keluar = spend adset ACTIVE.
8. **Kontak baru sejak cek sebelumnya** (bandingkan dengan cek terakhir di chat):
   | Adset | Kontak (sebelum → sekarang) | Spend sekarang | Spend nambah |
9. **Adset ≥2 jam tanpa kontak baru** (pakai riwayat cek di chat):
   | Adset | Spend | Kontak | CPL | Kontak terakhir masuk | Sejak | Spend sejak kontak terakhir |
10. **Kill candidate (SELALU PALING BAWAH)**: | Adset | Spend | Kontak | Aturan | —
    nama diawali 🔴. Kalau kosong tulis "belum ada" + 2–3 yang paling dekat batas.

### Aturan kill

- ABO budget <500rb: 0 kontak & spend ≥Rp75rb, atau 1 kontak & spend ≥Rp150rb.
- ABO budget ≥1jt: spend ≥Rp300rb & kontak ≤2.
- CBO: dinilai level campaign — 0 kontak & spend campaign ≥Rp150rb → pause campaign.
- Dikecualikan kalau hari itu punya purchase **DAN** kontak ≥1.

### Aturan penting

- **JANGAN pause/ubah apa pun tanpa perintah user.**
- Kalau disuruh pause: tarik data terbaru adset itu dulu. Kalau kontak/purchase
  nambah → lapor & tunggu. Kalau tidak nambah → pause (`{"status":"PAUSED"}`), lalu
  cek ulang `effective_status` dan lapor.
- ABO di-pause level adset. CBO di-pause level campaign.
- Jangan edit budget, targeting, nama, atau terbitkan iklan baru.
- Kalau user kasih perintah matiin pakai angka tanpa nyebut metriknya (misal
  "matiin yang di atas 75k"), jangan langsung eksekusi — tanya dulu spend atau CPL.
