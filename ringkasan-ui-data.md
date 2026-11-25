# Ringkasan UI & Data — Partlyfe (Sparepart Motor)

Ringkasan ini disusun dari `resources/views/`, `routes/web.php`, `app/Models/`, `database/seeders/`, ditambah controller dan migration (karena logika filter, harga, dan POS ada di sana). Tujuannya sebagai bahan konversi ke aplikasi demo statis 1 file untuk GitHub Pages.

## 0. Yang tidak sesuai asumsi awal

1. **UI Partlyfe tidak memakai Tailwind.** Tailwind v4 memang terpasang di `vite.config.js`, tetapi tidak ada view Partlyfe yang memanggil `@vite` atau class Tailwind. Semua gaya ditulis sebagai CSS biasa di `<style>` tiap layout dan view. Satu-satunya file yang memakai Tailwind adalah `resources/views/manul/index.html`, dan itu dashboard proyek lain ("Sakhanusa"), tidak dipanggil route mana pun, jadi tidak dimasukkan. Di bawah ada nilai hex asli ditambah *mapping* ke `tailwind.config` (buatan, tidak ada di proyek).
2. **Model `ProductUnit` tidak ada.** Satuan hanya kolom teks `products.unit` (default `PCS`), dan ke-30 produk memakai `PCS`. Tidak ada UI pilih satuan di katalog, detail, maupun POS. `code_template/Models/ProductUnit.php` hanya kelas kosong. Yang benar-benar ada adalah **harga 4 level** (`ProductPrice.price_level` 1–4), dan level itu dipilih **otomatis berdasarkan tier pelanggan**, bukan dipilih pengguna.
3. **Halaman Manajemen Kategori tidak ada.** Kategori hanya dibuat oleh seeder. Admin cuma memilih kategori lewat dropdown di form produk.
4. **Tidak ada satu pun file gambar di repo.** Seeder menyimpan 35 path gambar, tetapi tidak ada filenya di `public/`, `storage/`, maupun riwayat git. Akibatnya semua gambar produk tampil sebagai placeholder SVG.

Satu lagi: **tidak ada seeder untuk transaksi, keranjang, wishlist, atau broadcast.** Dengan data asli, dashboard admin, riwayat transaksi, dan notifikasi akan kosong.

---

## 1. Identitas Visual & Layout

### 1.1 Palet warna

| Token CSS | Hex | Dipakai untuk |
|---|---|---|
| `--black` | `#0A0A08` | Background `body` (customer & admin) |
| `--surface` | `#111110` | Sidebar admin, kartu, modal, input |
| `--surface-2` | `#181816` (customer) / `#1A1A16` (admin) | Hover kartu, section ringkasan |
| — | `#151513` | Kartu admin, stat card, header tabel |
| — | `#1A1A16` | Input auth, kotak gambar produk |
| — | `#0D0D0B` / `#0F0F0D` / `#141410` | Background section Home dan kartu kategori |
| — | `linear-gradient(145deg,#121210 0%,#0d0d0b 100%)` | Kartu cart, wishlist, transaksi, galeri |
| `--off-white` | `#F2EFE6` | Teks utama |
| `--cream` | `#E8E3D6` | Garis placeholder di Home |
| `--orange` | `#E8521A` | **Warna utama**: tombol, aksen, badge, harga |
| `--orange-dark` | `#C03F0E` | Teks tombol promo |
| `--orange-light` | `#FF6B35` | Hover tombol primary |
| `--gray-mid` | `#888880` | Teks redup, label |
| `--gray-light` | `#C8C4B8` | Teks sekunder, ikon navbar |
| `--green` | `#2ECC71` | Stok tersedia, sukses, "GRATIS" |
| `--red` | `#E74C3C` | Stok habis, error, wishlist aktif |
| border | `rgba(242,239,230,0.05–0.1)` | Semua garis tepi |

**Warna status transaksi (sisi customer, `.tx-status.*`):**

| Status | Warna teks | Background |
|---|---|---|
| pending | `#F1C40F` | `rgba(241,196,15,.15)` |
| processing | `#3498DB` | `rgba(52,152,219,.15)` |
| shipped | `#9B59B6` | `rgba(155,89,182,.15)` |
| delivered | `#2ECC71` | `rgba(46,204,113,.15)` |
| done | `#6c757d` | `rgba(108,117,125,.15)` |
| cancelled | `#E74C3C` | `rgba(231,76,60,.15)` |

**Warna status pesanan (sisi admin, `.order-status-badge.status-*`):** pending `#ffc107`, processing `#007bff`, shipped `#28a745`, delivered `#2ecc71`, done `#6c757d`, cancelled `#dc3545`. Background-nya warna yang sama dengan alpha .2.

**Tipe notifikasi (`.bc-type.*`):** promo `#E8521A`, system `#3498DB`, info `#2ECC71`, cart `#9B59B6`, wishlist `#F1C40F` (background alpha .15).

**Badge stok (admin):** habis `#dc3545`, ≤5 `#ffc107`, aman `#28a745`.

**Aksen dashboard admin:** biru `#3b82f6`, hijau `#22c55e`, merah `#ef4444`, kuning `#f59e0b`, ungu `#a855f7`. Palet doughnut chart: `['#E8521A','#f59e0b','#3b82f6','#22c55e','#a855f7']`.

**Mapping ke Tailwind (opsional, buatan sendiri):**
```js
tailwind.config = { theme: { extend: {
  colors: {
    ink:'#0A0A08', surface:'#111110', surface2:'#181816', panel:'#151513', field:'#1A1A16',
    offwhite:'#F2EFE6', cream:'#E8E3D6', muted:'#888880', soft:'#C8C4B8',
    brand:{ DEFAULT:'#E8521A', dark:'#C03F0E', light:'#FF6B35' },
    ok:'#2ECC71', danger:'#E74C3C'
  },
  fontFamily: {
    display:['"Bebas Neue"','sans-serif'], body:['"DM Sans"','sans-serif'], mono:['"Space Mono"','monospace']
  }
}}}
```

### 1.2 Font

```html
<link href="https://fonts.googleapis.com/css2?family=Bebas+Neue&family=DM+Sans:wght@300;400;500;600&family=Space+Mono:wght@400;700&display=swap" rel="stylesheet">
```
- **Bebas Neue** (`--font-display`): judul, logo, harga besar, angka statistik.
- **DM Sans** 300–600 (`--font-body`): teks isi, 15px, line-height 1.6.
- **Space Mono** 400/700 (`--font-mono`): label kecil uppercase, SKU, badge, breadcrumb, harga kecil.
- Layout admin **tidak memuat Google Fonts** dan body-nya `system-ui`. Jadi Bebas Neue dan Space Mono di admin jatuh ke font fallback. Kalau demo 1 file memuat font secara global, tampilan admin akan sedikit lebih rapi daripada aslinya.
- Font Instrument Sans (bawaan Vite) tidak dipakai di mana pun.

### 1.3 Struktur navigasi

**Navbar customer** (`layouts/app.blade.php`): sticky, tinggi 64px, background `rgba(10,10,8,.92)` dengan `backdrop-filter: blur(16px)`, padding `0 4rem`. Isinya dari kiri ke kanan:
1. Logo teks `PART<span>L</span>YFE` (Bebas 1.7rem, huruf **L** oranye), link ke `/`.
2. Kotak pencarian (maks. 440px, disembunyikan di bawah 768px), placeholder "Cari sparepart, merek, tipe motor...", tombol kaca pembesar oranye, dan dropdown hasil live.
3. Spacer.
4. Link teks **SHOP** (oranye, mono) ke `/produk`.
5. Kalau sudah login: 🔔 lonceng (dropdown notifikasi) → ♡ wishlist (badge jumlah) → 🛒 keranjang (badge total qty, "9+" kalau >9) → avatar bulat berisi inisial nama, link ke `/akun`.
6. Kalau tamu: ikon 👤 ke `/login`.

**Footer**: satu baris `.footer-mini`: "© {tahun} PARTLYFE — Sparepart Motor Terpercaya Indonesia" (mono, 0.75rem, abu, rata tengah).

**Widget melayang**: tombol bulat AI Chat 60px di kanan bawah (gradient `#E8521A→#b3390f`), muncul di semua halaman customer.

**Admin** (`layouts/admin.blade.php`): `body` flex setinggi 100vh.
- Sidebar 250px (`#111110`): brand "PARTLYFE ADMIN" (oranye, bold 1.5rem), lalu 7 menu: Analytics Dashboard, Manajemen Produk, POS Kasir, Manajemen Pesanan, Kirim Broadcast, Manajemen Pengguna, Laporan. Menu aktif/hover: background `rgba(232,82,26,.1)`, teks oranye, `border-right: 3px solid #E8521A`.
- Header: "Admin Control Panel" dan tombol outline "Logout Akun".
- Konten: padding 2rem, bisa di-scroll.
- `/admin/tiers` dan `/admin/pos/stock-in` punya route, tetapi **tidak ada di sidebar**.

**File yang tidak dipakai** (prototipe hardcoded, tidak dipanggil route): `layouts/login`, `layouts/dashboard`, `layouts/products`, `layouts/broadcast`, `layouts/pos`, `components/navbar`, `components/footer`.

**Breakpoint**: 1024px (grid jadi 2 kolom, sidebar katalog disembunyikan), 768px (search navbar disembunyikan, layout detail/cart/checkout/akun jadi 1 kolom), 640px (grid Home 1 kolom), 480px (grid katalog 1 kolom).

### 1.4 Potongan komponen utama (CSS/HTML asli, sudah dirapatkan)

**Dasar: variabel, tombol, badge, judul section**
```css
:root{--black:#0A0A08;--off-white:#F2EFE6;--cream:#E8E3D6;--surface:#111110;--surface-2:#181816;
--orange:#E8521A;--orange-dark:#C03F0E;--orange-light:#FF6B35;--gray-mid:#888880;--gray-light:#C8C4B8;
--green:#2ECC71;--red:#E74C3C;--font-display:'Bebas Neue',sans-serif;--font-body:'DM Sans',sans-serif;--font-mono:'Space Mono',monospace}
*,*::before,*::after{margin:0;padding:0;box-sizing:border-box}
body{background:var(--black);color:var(--off-white);font-family:var(--font-body);font-size:15px;line-height:1.6}
a{color:inherit;text-decoration:none}
.page-wrapper{max-width:1320px;margin:0 auto;padding:2rem 4rem 4rem}
.breadcrumb{display:flex;align-items:center;gap:.4rem;font-size:.75rem;color:var(--gray-mid);margin-bottom:2rem;font-family:var(--font-mono);letter-spacing:.04em;flex-wrap:wrap}
.breadcrumb span{color:var(--orange)}
.btn{display:inline-flex;align-items:center;gap:.5rem;padding:.65rem 1.5rem;border-radius:2px;font-size:.87rem;font-weight:600;letter-spacing:.04em;cursor:pointer;border:none;transition:all .2s}
.btn-primary{background:var(--orange);color:var(--off-white)}.btn-primary:hover{background:var(--orange-light)}
.btn-outline{background:transparent;color:var(--off-white);border:1px solid rgba(242,239,230,.2)}
.btn-outline:hover{border-color:var(--off-white);background:rgba(242,239,230,.05)}
.btn-sm{padding:.45rem 1rem;font-size:.78rem}.btn-lg{padding:.9rem 2.5rem;font-size:.95rem}
.badge{display:inline-flex;align-items:center;padding:.2rem .6rem;border-radius:2px;font-family:var(--font-mono);font-size:.62rem;letter-spacing:.08em;text-transform:uppercase;font-weight:700}
.badge-orange{background:var(--orange);color:var(--off-white)}
.badge-green{background:rgba(46,204,113,.15);color:var(--green);border:1px solid rgba(46,204,113,.2)}
.sec-label{display:inline-flex;align-items:center;gap:.5rem;font-family:var(--font-mono);font-size:.65rem;letter-spacing:.12em;text-transform:uppercase;color:var(--orange);margin-bottom:.75rem}
.sec-label::before{content:'';width:16px;height:1px;background:var(--orange)}
.sec-title{font-family:var(--font-display);font-size:2.2rem;letter-spacing:.03em;line-height:1;margin-bottom:2rem}
```

**Header / navbar**
```html
<nav class="nav">
  <a href="#/" class="nav-logo">PART<span>L</span>YFE</a>
  <div class="nav-search" style="position:relative">
    <input id="searchInput" placeholder="Cari sparepart, merek, tipe motor..." autocomplete="off">
    <button class="nav-search-btn"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round"><circle cx="11" cy="11" r="8"/><path d="m21 21-4.35-4.35"/></svg></button>
    <div id="searchResults" class="search-results"></div>
  </div>
  <div class="nav-spacer"></div>
  <div class="nav-icons">
    <a href="#/produk" class="nav-icon-btn" style="font-size:.8rem;font-family:var(--font-mono);font-weight:bold;color:var(--orange)">SHOP</a>
    <!-- auth: bell, heart + <span class="nav-badge">3</span>, cart + badge, <a class="nav-avatar">U</a> -->
  </div>
</nav>
```
```css
.nav{position:sticky;top:0;z-index:200;height:64px;background:rgba(10,10,8,.92);backdrop-filter:blur(16px);border-bottom:1px solid rgba(242,239,230,.07);display:flex;align-items:center;padding:0 4rem;gap:1.5rem}
.nav-logo{font-family:var(--font-display);font-size:1.7rem;letter-spacing:.04em}.nav-logo span{color:var(--orange)}
.nav-search{flex:1;max-width:440px;display:flex;background:var(--surface);border:1px solid rgba(242,239,230,.1);border-radius:3px;overflow:hidden}
.nav-search input{flex:1;padding:.55rem 1rem;background:transparent;border:none;color:var(--off-white);font-size:.85rem;outline:none}
.nav-search-btn{padding:.55rem 1rem;background:var(--orange);border:none;color:var(--off-white)}.nav-search-btn svg{width:16px;height:16px}
.nav-spacer{flex:1}.nav-icons{display:flex;align-items:center;gap:.25rem}
.nav-icon-btn{width:40px;height:40px;border-radius:3px;background:transparent;border:none;color:var(--gray-light);display:flex;align-items:center;justify-content:center;position:relative}
.nav-icon-btn:hover{background:var(--surface);color:var(--off-white)}.nav-icon-btn svg{width:20px;height:20px}
.nav-badge{position:absolute;top:4px;right:4px;width:16px;height:16px;border-radius:50%;background:var(--orange);font-size:9px;font-weight:700;font-family:var(--font-mono);display:flex;align-items:center;justify-content:center}
.nav-avatar{width:34px;height:34px;border-radius:50%;background:#2A2A26;border:1.5px solid rgba(232,82,26,.4);display:flex;align-items:center;justify-content:center;font-family:var(--font-mono);font-size:.65rem;font-weight:700;color:var(--orange)}
.notif-dropdown{position:absolute;top:120%;right:0;width:340px;background:linear-gradient(145deg,#121210,#0d0d0b);border:1px solid rgba(242,239,230,.08);border-radius:4px;box-shadow:0 15px 35px rgba(0,0,0,.6);display:none;z-index:250}
.notif-dropdown.show{display:block}
.unread-dot{width:6px;height:6px;background:var(--orange);border-radius:50%;position:absolute;top:1.25rem;right:1rem;box-shadow:0 0 8px var(--orange)}
```
> Catatan: class `.search-results` / `.search-item` tidak punya CSS sama sekali di proyek asli, jadi dropdown pencarian harus dibuatkan gayanya sendiri.

**Kartu produk katalog**
```html
<a href="#/produk/1" class="prod-card">
  <div class="prod-img"><!-- <img> atau placeholder SVG --></div>
  <div class="prod-body">
    <div class="prod-brand">AHM</div>
    <div class="prod-name">Kampas Rem Depan Vario / Beat FI</div>
    <div class="prod-spec">Stok: 50 PCS</div>
    <div class="prod-rating"><span class="s">★</span> 4.8</div>
    <div class="prod-footer-price">
      <div><!-- <div class="prod-price-old">Rp 55.000</div> hanya jika tier > 1 -->
        <div class="prod-price">Rp 55.000</div></div>
      <button class="prod-add-btn"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" width="16"><path d="M12 5v14M5 12h14"/></svg></button>
    </div>
  </div>
</a>
```
```css
.product-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:1rem}
.prod-card{background:var(--surface);display:flex;flex-direction:column;border-radius:4px;border:1px solid rgba(242,239,230,.03);transition:background .2s}
.prod-card:hover{background:var(--surface-2)}
.prod-img{aspect-ratio:1;max-height:250px;background:#1A1A16;display:flex;align-items:center;justify-content:center;padding:.5rem}
.prod-img img{width:100%;height:100%;object-fit:contain}
.prod-body{padding:1rem;display:flex;flex-direction:column;flex:1}
.prod-brand{font-family:var(--font-mono);font-size:.6rem;color:var(--orange);margin-bottom:.3rem;text-transform:uppercase}
.prod-name{font-size:.87rem;font-weight:500;line-height:1.45;margin-bottom:.4rem}
.prod-spec,.prod-rating{font-size:.75rem;color:var(--gray-mid);margin-bottom:.75rem}.prod-rating .s{color:var(--orange)}
.prod-footer-price{margin-top:auto;display:flex;align-items:flex-end;justify-content:space-between}
.prod-price{font-family:var(--font-display);font-size:1.35rem}
.prod-price-old{font-size:.72rem;color:var(--gray-mid);text-decoration:line-through}
.prod-add-btn{width:34px;height:34px;background:transparent;border:1px solid rgba(242,239,230,.2);border-radius:2px;color:var(--off-white);display:flex;align-items:center;justify-content:center}
```

**Kartu produk unggulan (Home), dengan garis oranye muncul di atas saat hover**
```css
.produk-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:1px}
.produk-card{background:#0F0F0D;padding:1.75rem;position:relative;overflow:hidden;display:flex;flex-direction:column;transition:background .25s}
.produk-card:hover{background:#171713}
.produk-card::after{content:'';position:absolute;top:0;left:0;right:0;height:2px;background:var(--orange);transform:scaleX(0);transform-origin:left;transition:transform .3s}
.produk-card:hover::after{transform:scaleX(1)}
.produk-img-wrap{aspect-ratio:1;background:#1A1A16;border-radius:2px;display:flex;align-items:center;justify-content:center;margin-bottom:1.25rem;position:relative}
.produk-badge{position:absolute;top:.75rem;left:.75rem;background:var(--orange);font-family:var(--font-mono);font-size:.6rem;letter-spacing:.08em;padding:.25rem .6rem;text-transform:uppercase}
.produk-price{font-family:var(--font-display);font-size:1.5rem}
.produk-add{width:36px;height:36px;border:1px solid rgba(242,239,230,.2);border-radius:50%;background:transparent}
.produk-add:hover{background:var(--orange);border-color:var(--orange)}
```
> Di Blade aslinya, elemen kartu Home ditulis dengan class `prod-card`, padahal CSS `.prod-card` hanya dimuat di halaman katalog. Akibatnya di Home kartu tampil tanpa background dan efek hover. Gunakan `.produk-card` di demo.

**Blok harga (halaman detail)**
```css
.price-plate{background:#111110;border:1px solid rgba(242,239,230,.04);border-left:3px solid var(--orange);padding:1.5rem;border-radius:4px;margin-bottom:2rem;box-shadow:inset 0 0 20px rgba(0,0,0,.2)}
.price-was{font-family:var(--font-mono);font-size:.85rem;color:var(--gray-mid);text-decoration:line-through}
.price-now{font-family:var(--font-display);font-size:2.8rem;line-height:1}
.prod-main-title{font-family:var(--font-display);font-size:2.5rem;line-height:1.1;letter-spacing:.02em}
.prod-sku-tag{font-family:var(--font-mono);font-size:.72rem;letter-spacing:.1em;text-transform:uppercase;color:var(--orange)}
```

**Badge status, tab filter, dan kartu admin**
```css
.tx-status{font-family:var(--font-mono);font-size:.72rem;font-weight:700;text-transform:uppercase;padding:.3rem .7rem;border-radius:2px}
.bc-type{font-family:var(--font-mono);font-size:.65rem;font-weight:700;letter-spacing:.1em;text-transform:uppercase;padding:.2rem .5rem;border-radius:2px}
.filter-tab{padding:.5rem 1rem;border-radius:2px;font-size:.82rem;border:1px solid rgba(242,239,230,.1);background:var(--surface);color:var(--gray-light)}
.filter-tab.active{background:var(--orange);border-color:var(--orange);color:#fff}
/* admin */
.admin-card{background:#151513;border:1px solid rgba(242,239,230,.08);border-radius:8px;padding:2rem;box-shadow:0 10px 30px -10px rgba(0,0,0,.5)}
.admin-table th{background:#1a1a18;color:#888880;padding:1rem 1.5rem;font-size:.8rem;text-transform:uppercase;letter-spacing:.05em}
.admin-table td{padding:1.25rem 1rem;border-bottom:1px solid rgba(242,239,230,.04)}
.admin-table tbody tr:hover{background:rgba(232,82,26,.03)}
.stat-card{background:#151513;border:1px solid rgba(242,239,230,.08);border-radius:6px;padding:1.5rem;cursor:pointer;position:relative;overflow:hidden;transition:all .3s}
.stat-card:hover{transform:translateY(-5px);border-color:rgba(232,82,26,.4)}
.stat-card.active{border-color:#E8521A;background:linear-gradient(145deg,#1a1512,#151513)}
.stat-card.active::before{content:'';position:absolute;top:0;left:0;right:0;height:3px;background:#E8521A}
.stat-title{color:#888880;font-size:.85rem;text-transform:uppercase;font-family:var(--font-mono)}
.stat-value{font-size:2.5rem;font-family:var(--font-display);line-height:1}
.modal{display:none;position:fixed;inset:0;background:rgba(0,0,0,.8);z-index:1000;align-items:center;justify-content:center}.modal.active{display:flex}
.modal-content{background:#111110;border:1px solid rgba(242,239,230,.1);padding:2rem;border-radius:8px;width:90%;max-width:500px}
.form-control{width:100%;padding:.75rem;background:#111110;border:1px solid rgba(242,239,230,.08);border-radius:4px;color:#F2EFE6}
.form-control:focus{outline:none;border-color:#E8521A;box-shadow:0 0 0 2px rgba(232,82,26,.15)}
```

**Toast**: SweetAlert2 dari CDN (`toast:true, position:'top-end', timer:3000, background:'#151513', color:'#fff', iconColor:'#E8521A'`). Beberapa halaman (cart, wishlist, checkout, produk admin) memakai toast buatan sendiri: kotak `#111110` dengan `border-left` 3–4px hijau (sukses) atau merah (gagal), di `top:80px; right:24px`.

---

## 2. Halaman & Alur Interaksi

### 2.1 Peta route

| Sisi | URL | Halaman | Login? |
|---|---|---|---|
| Customer | `/`, `/promo` | Beranda (`/promo` hanya menampilkan Beranda yang sama) | – |
| | `/produk`, `/kategori` | Katalog | – |
| | `/kategori/{slug}` | Katalog dengan filter kategori | – |
| | `/produk/{id}` | Detail produk | – |
| | `/cart` | Keranjang | ✔ |
| | `/wishlist` | Wishlist | ✔ |
| | `/checkout` | Checkout | ✔ |
| | `/transactions`, `/transactions/{invoice}` | Riwayat transaksi, Invoice | ✔ |
| | `/notifikasi` | Notifikasi (broadcast/promo) | ✔ |
| | `/akun` | Profil | ✔ |
| | `/login`, `/register`, `/forgot-password`, `/reset-password/{token}` | Autentikasi | – |
| | `/dashboard` | Katalog versi lain (**error**, lihat §5) | ✔ |
| Admin | `/admin/dashboard` | Analytics Dashboard | admin |
| | `/admin/products` | Produk & stok | admin |
| | `/admin/pos` | POS Kasir | admin |
| | `/admin/pos/stock-in` | Form stok masuk (tidak ada di sidebar) | admin |
| | `/admin/orders`, `/admin/orders/{id}` | Daftar & detail pesanan | admin |
| | `/admin/users` | Pengguna & level harga | admin |
| | `/admin/tiers` | Level harga (tidak ada di sidebar; **error**) | admin |
| | `/admin/broadcasts` | Kirim broadcast | admin |
| | `/admin/reports` | Laporan | admin |

### 2.2 Logika harga (berlaku di semua halaman)

- Setiap produk punya:
  - `base_price` = **harga modal** = 80% × harga eceran.
  - 4 baris `ProductPrice`: L1 = eceran, L2 = 90%, L3 = 85%, L4 = 82%.
- **Storefront** (katalog, detail, cart, checkout, wishlist):
  - `tierPriceLevel` = `price_level` dari tier aktif pelanggan. Tier aktif = `CustomerTierAssignment` terbaru yang `expires_at`-nya kosong atau belum lewat. Tamu dan pelanggan tanpa tier memakai level 1.
  - Harga tampil = harga di level itu; kalau tidak ada, pakai `base_price`.
  - Kalau level > 1, harga level 1 ditampilkan dicoret di atasnya.
- **POS** memakai sistem yang berbeda: kolom `users.customer_group` (`poin`, `level1`–`level4`), dipetakan oleh `PosService::mapGroupToLevel` (`poin` → 1).
- Format uang di semua tempat: `Rp 55.000` (`number_format(x,0,',','.')` atau `toLocaleString('id-ID')`).

### 2.3 Halaman customer

**A. Beranda (`/`)**
- **Hero** (2 kolom, 100vh):
  - Teks latar raksasa "MOTOR" oranye dengan opasitas 4%.
  - Eyebrow "Toko Sparepart Motor #1 Indonesia".
  - Judul "SPAREPART / MOTOR / **TERBAIK.**" (kata terakhir oranye).
  - Subjudul "Dari mesin hingga bodi — semua tersedia. Suku cadang original & aftermarket berkualitas untuk semua merek motor favoritmu."
  - Tombol "Belanja Sekarang →" ke `/produk`, link "Lihat Kategori" ke `/kategori`.
  - Statistik: `{jumlah produk}+` "Produk Tersedia" (30+), "200K+" "Pelanggan Puas", "99%" "Produk Original".
  - Kolom kanan: ilustrasi mesin SVG, badge "Original & Bergaransi", ticker "Merek Tersedia" (Honda, Yamaha, Suzuki, Kawasaki, TVS).
- **Marquee** (animasi 20 detik, diulang 2×): ORIGINAL PARTS ✦ FAST DELIVERY ✦ GARANSI RESMI ✦ 50.000+ PRODUK ✦ HARGA TERBAIK ✦ SPAREPART MOTOR ✦ PARTLYFE ✦
- **Kategori Produk**: label "Jelajahi" + judul "KATEGORI PRODUK" + tombol "Lihat Semua →".
  - Grid 4 kolom berisi **4 kategori pertama** (Pengereman, Kelistrikan, Kaki-Kaki & Roda, Mesin & Penggerak).
  - Kartu rasio 3:4, ikon SVG di tengah (opasitas .12), nama (Bebas 1.6rem), "{n} produk".
  - Saat hover: gradient oranye dari bawah dan panah ↗ muncul.
  - Klik membuka `/kategori/{slug}`.
- **Produk Unggulan**: label "Terlaris Minggu Ini" + "PRODUK UNGGULAN".
  - Berisi **3 produk acak yang stoknya > 0**. Badge "Terlaris" di kartu pertama, "New" di kartu ketiga.
  - Tiap kartu: brand, gambar, nama, "Stok: N PCS", harga L1.
  - Tombol ＋ tidak melakukan apa-apa (`preventDefault`).
- **Banner promo** (background oranye penuh, teks latar "SALE"):
  - Eyebrow "Penawaran Terbatas · Berakhir Minggu Ini".
  - Judul "GANTI OLI / HEMAT 30% / BULAN INI".
  - Teks "Oli mesin, filter, dan semua produk perawatan berkurang harga. Jaga performa motormu tanpa perlu khawatir dengan harga."
  - Tombol putih "Klaim Promo Sekarang →" ke `/produk`.
  - Diskon 30% ini hanya teks; tidak ada logika diskonnya.
- CSS untuk section testimoni, keunggulan, merek, dan newsletter ada di layout, tetapi **tidak dirender**.

**B. Katalog (`/produk`)**
- Breadcrumb "HOME / KATALOG". Layout 2 kolom: sidebar 260px + area produk.
- **Filter kategori**:
  - Daftar "Semua Produk (30)" lalu tiap kategori dengan jumlah produknya.
  - Item aktif: background oranye 12%, teks oranye.
  - Klik mengubah `?category={id}`; parameter lain dipertahankan, kecuali `page`.
- **Rentang harga**:
  - Dua slider range ditumpuk (step 1000, min/max dari `base_price` = Rp 13.600 – Rp 272.000).
  - Label angka ikut berubah saat digeser; slider min tidak bisa melewati max.
  - Tombol "Terapkan" mengirim GET `?min_price&max_price` (plus `search`/`category` kalau ada). "Reset" kembali ke `/produk`.
- **Pencarian**: `?search=` mencocokkan `name` atau `brand` (LIKE). Diisi dari kotak search navbar (Enter atau klik tombol).
- **Search live di navbar**:
  - Jalan setelah ≥2 karakter, debounce 300ms, maksimal 8 hasil.
  - Tiap hasil: thumbnail, nama, brand, harga. Klik membuka detail.
  - Klik di luar dropdown menutupnya.
- Topbar "Menampilkan X–Y dari Z produk". Grid 3 kolom, **9 produk per halaman**, ada pagination.
- Rating "★ 4.8" di semua kartu adalah nilai tetap (hardcoded). Tombol ＋ hanya bagian dari link ke detail.

**C. Detail Produk (`/produk/{id}`)**
- Tombol oranye "← KEMBALI KE PRODUK" dan breadcrumb "HOME / SPARES / {NAMA PRODUK}".
- **Kiri, galeri** (sticky):
  - Kotak utama rasio 1:1 dengan badge "TERLARIS" (selalu tampil).
  - Baris 4 thumbnail; thumbnail pertama aktif dengan border oranye.
  - Thumbnail **tidak bisa diklik** (tidak ada JS-nya).
- **Kanan**:
  - "{BRAND} · SKU: {item_code}", lalu judul (Bebas 2.5rem).
  - Baris meta "⭐ 0.0 | 0 Ulasan Verified | Terjual 0 Unit". Ketiga kolom ini tidak ada di database, jadi selalu 0.
  - Blok harga sesuai tier.
  - Bar stok "Stok Sistem Gudang: **N unit** tersisa." dengan titik berdenyut; titik diberi class `low-stock` kalau stok < 10.
  - **Kuantitas**: tombol − / input / +, rentang 1 sampai stok, teks "Max pembelian: N unit".
  - Tombol **"Tambah ke Keranjang"**: AJAX POST, lalu toast "Berhasil ditambahkan ke keranjang!". Server menolak kalau qty di keranjang ditambah qty baru melebihi stok.
  - Tombol **"⚡ Beli Sekarang"**: POST `/buy-now/{id}?qty=`. Server **menghapus semua item lain di keranjang**, mengisi keranjang hanya dengan produk ini sebanyak qty, lalu redirect ke `/checkout`.
  - Tombol **♡** (hanya kalau login): toggle wishlist lalu reload. Hati merah terisi kalau produk sudah di wishlist.
  - Tabel spesifikasi: Manufaktur / Material Pad / Rekomendasi Batas Pakai / Berat Total. Kolomnya tidak ada di database, jadi semua "-".
- **"Produk Serupa Lainnya"**: maksimal 5 produk acak dari kategori yang sama dan stoknya > 0, grid `auto-fill minmax(220px,1fr)`. Tombol keranjang kecil menambah 1 unit (mengikuti qty di stepper halaman).

**D. Keranjang (`/cart`)**
- Judul "SHOPPING CART" + badge "N ITEMS".
- Kosong: ikon, "KERANJANG KOSONG", "Belum ada produk di keranjang Anda.", tombol "Lihat Katalog Produk".
- Baris "☑ Pilih Semua Barang" dan checkbox per item hanya hiasan, tidak berfungsi.
- **Kartu item** (grid `auto 100px 1fr`): checkbox, gambar, brand, nama, harga sesuai tier (harga L1 dicoret kalau tier > 1), peringatan merah "⚠ Stok tersisa: N pcs" kalau qty > stok, stepper − qty + (tombol − nonaktif di qty 1, + nonaktif di qty = stok), tombol hapus 🗑 (`confirm()`, lalu animasi fade + geser ke kiri).
- **Ringkasan** (sticky): "Total Harga (N Barang)", "Ongkos Kirim **GRATIS**" (hijau), pil "Aman! Belanja di atas Rp 200.000, otomatis gratis ongkir reguler." (padahal ongkir selalu 0), "Total Harga", tombol "Lanjut ke Checkout".

**E. Wishlist (`/wishlist`)**
- Judul "MY WISHLIST" + "N ITEMS SAVED". Kosong: "WISHLIST KOSONG" + tombol "Jelajahi Produk".
- Grid kartu: tombol × di pojok (modal konfirmasi buatan sendiri "Hapus produk ini dari wishlist?" dengan tombol Batal / Ya, Hapus), gambar, brand, nama, status "● Stok Tersedia (N pcs)" (hijau) atau "● Stok Habis" (merah), harga sesuai tier, tombol 🛒 tambah 1 ke keranjang (hanya kalau stok ada).

**F. Checkout (`/checkout`)**
- Kiri:
  - **ALAMAT PENGIRIMAN**: kartu dengan border oranye berisi nama, HP, alamat dari profil. Kalau alamat kosong: "Alamat belum diatur. Harap tambahkan alamat pada profil."
  - **DETAIL PEMBELIAN**: per baris nama, harga (dicoret kalau tier > 1) × qty, subtotal.
  - Banner "Level Harga N berlaku (otomatis diterapkan)" kalau tier > 1.
  - **METODE PEMBAYARAN**: satu opsi saja, "Transfer Bank / E-Wallet (Midtrans)".
- Kanan, **RINGKASAN ORDER**:
  - Kalau tier > 1: Subtotal asli dicoret dan "Hemat (Level N) −Rp X" (hijau).
  - "Total Items (N barang)", **Total** (oranye), tombol "Bayar Sekarang".
- Sumber item: query `?product_id&qty`, atau kalau tidak ada, seluruh isi keranjang. Kalau keranjang kosong, redirect ke cart.
- Alur bayar:
  1. POST `/checkout/initiate` membuat Transaction `TRX-{timestamp}-{userId}` berstatus `pending`, menyimpan detail, dan **mengosongkan keranjang**.
  2. Popup Midtrans Snap muncul.
  3. Sukses/pending: toast, lalu pindah ke `/transactions` setelah 1,5 detik.
  4. Callback `settlement`/`capture`: status jadi `processing` dan stok dikurangi. `cancel`/`expire`/`deny`: status jadi `cancelled`.

**G. Riwayat Transaksi (`/transactions`)**
- Judul "RIWAYAT TRANSAKSI".
- Tab: **Semua** · **Menunggu** (pending) · **Diproses** (processing) · **Gagal/Dibatalkan** (cancelled). Status shipped/delivered/done hanya muncul di tab "Semua".
- Kartu transaksi (grid `1fr auto auto auto`): nomor invoice + tanggal "d M Y, H:i" · "N Produk / N unit total" · badge status · total. Klik membuka invoice. 10 per halaman.
- Kosong: "BELUM ADA TRANSAKSI" + tombol "Mulai Belanja".
- **Invoice**:
  - Header "PARTLYFE / Distributor Sparepart Motor Terpercaya" dan "INVOICE {nomor}".
  - Badge status: **LUNAS** (hijau) untuk processing/shipped/delivered/done, **DIBATALKAN** (merah) untuk cancelled, **PENDING** (oranye) untuk lainnya.
  - "Ditagihkan Kepada" (nama, HP, email) dan tanggal dalam WIB.
  - Tabel Nama Barang / Qty / Harga / Subtotal, lalu Subtotal dan **TOTAL** dengan garis oranye di atasnya.
  - Penutup: "Terima kasih telah berbelanja di Partlyfe. Simpan nota ini sebagai bukti transaksi Anda yang sah."

**H. Notifikasi / Broadcast (`/notifikasi`)**
- Judul "NOTIFIKASI" + tombol "Tandai Semua Dibaca".
- Kartu: badge tipe (PROMO/SYSTEM/INFO/CART/WISHLIST), tanggal, judul, pesan. **Membuka halaman ini otomatis menandai semua notifikasi sebagai dibaca.**
- **Dropdown di lonceng navbar**: 5 notifikasi belum dibaca. Ikon per tipe: promo = label harga, system = info, lainnya = centang. Lalu judul, 50 karakter pertama pesan, waktu relatif, titik oranye. Klik item menandai dibaca lalu reload. Header "NOTIFIKASI" + "LIHAT SEMUA". Data di-poll setiap 30 detik.
- **Notifikasi yang dibuat otomatis untuk pelanggan**:
  - "Item Ditambahkan ke Keranjang" (cart)
  - "Ditambahkan ke Wishlist" (wishlist) / "Dihapus dari Wishlist" (info)
  - "Beli Langsung" (system)
  - "Pembayaran Berhasil" (system)
  - "Status Pesanan Diperbarui" (system)
  - "Level Harga Diperbarui" (promo)
  - Broadcast dari admin

**I. Akun (`/akun`)**
- Sidebar: avatar huruf inisial dengan centang verified, nama, "ID: PL-{id}-2026", menu Profile Settings · My Orders · My Wishlist · Log Out.
- Judul "ACCOUNT SETTINGS".
- Kartu "Informasi Kontak": Nama Lengkap, Nomor Telepon (badge "Verified"), Email (nonaktif, badge "Primary"), Alamat Lengkap, tombol "Simpan Perubahan".
- Kartu "Ringkasan Akun": Total Transaksi (selalu 0, variabelnya tidak dikirim controller), Produk Wishlist, "Keamanan Akun: ✓ Password terlindungi", link "Ubah Kata Sandi →" ke halaman lupa password.

**J. Autentikasi** (memakai layout customer, navbar tetap tampil)
- **Login**: "MASUK **PARTLYFE**", Email, Password, tombol "MASUK SISTEM", link "Lupa Password?" dan "Belum punya akun? Daftar Sekarang". Admin diarahkan ke `/admin/dashboard`, customer ke halaman sebelumnya atau `/`. Error: "Email atau password salah."
- **Register**: "DAFTAR **PARTLYFE**", Nama Lengkap, Email, Password (min. 6), Konfirmasi Password, No. HP (opsional), tombol "BUAT AKUN". Langsung login, lalu ke `/`.
- **Lupa/Reset password**: form email (perlu mail server), lalu form password baru.

**K. Widget AI Chat "Mekanik AI Pro"** (ada di semua halaman customer)
- Tombol melayang membuka modal (lebar maks. 500px, tinggi 65vh, border oranye 30%).
- Pesan pembuka: "Halo! Saya Mekanik AI dari Partlyfe. Motormu lagi ada masalah apa nih? Silakan ketik keluhanmu di bawah ini."
- Input "Ketik keluhan motormu..." dan tombol "Kirim" (atau Enter). Bubble pengguna di kanan (oranye 15%), lalu "AI sedang mengetik...", lalu balasan dan sampai 3 kartu produk (gambar 60px, nama + badge Tersedia/Habis, "brand · SKU: kode", harga oranye). Klik kartu membuka detail.
- **Mode offline (yang perlu ditiru di demo statis)**:
  - Kalau pesan mengandung "oli"/"mpx": "Untuk produk oli mesin terbaik di PartLyfe, kami sangat merekomendasikan varian oli AHM MPX asli Honda untuk menjaga performa mesin tetap stabil dan irit, Bos!" + produk #25 dan #26.
  - Kalau mengandung "rem"/"kampas": "Safety nomor satu, Bos! Kampas rem kita punya kualitas cengkeraman pakem dan tahan panas." + produk #1 dan #2.
  - Selain itu: "Halo Bos! Di PartLyfe kita menyediakan berbagai sparepart roda dua premium. Kamu lagi cari komponen spesifik apa nih buat motor kesayanganmu? Biar Mechanix bantu cek stoknya! 🛠️" (di sumber aslinya emoji ini rusak jadi `ЁЯЫая╕П`).

### 2.4 Halaman admin

**A. Analytics Dashboard**
- Judul "Analytics Dashboard" + petunjuk "Geser ikon titik (⋮⋮) untuk mengatur tata letak widget."
- **4 stat card yang berfungsi sebagai tab**. Klik satu kartu menampilkan tabel di bawahnya.

| Kartu | Ikon/warna | Isi tabel |
|---|---|---|
| Pesanan Baru (jumlah pending) | 📦 biru | 5 pesanan pending: ID `ORD-001`, Pelanggan, Total, Status |
| Pendapatan Hari Ini (processing–done hari ini) | 💰 hijau | Judulnya "Rincian Transaksi Selesai Hari Ini", isinya pendapatan **per bulan** |
| Stok Menipis (stok 1–5 ditambah stok 0) | ⚠️ merah, "Produk butuh re-stock segera!" | Kode SKU, Nama, Sisa Stok, tombol "+ Tambah" (modal tambah stok cepat) |
| Pelanggan Baru (7 hari terakhir) | 👥 oranye | Tanggal daftar (relatif), Nama, Email, Total Belanja (selalu "Rp 0") |

- **Analitik Tambahan**: grid widget yang bisa diurutkan dengan SortableJS (urutan disimpan di `localStorage['partlyfe_dashboard_layout']`) dan tombol "⚙️ Atur Widget" (toggle tampil/sembunyi, disimpan di `localStorage['partlyfe_dashboard_visibility']`).
  - "📈 Grafik Arus Kas": Chart.js bar, `#22c55e`, 6 bulan.
  - "🎯 Penjualan per Kategori": doughnut, `cutout 70%`, top 5.
  - "💡 Info Server & Sistem": Total Produk, Total Customer, Total Pesanan.
- Kondisi dengan data seeder: Pesanan 0, Pendapatan Rp 0, Stok Menipis 3 (PRT-023, PRT-024, PRT-030 yang stoknya 0), Pelanggan Baru 2, grafik "Belum ada data...".

**B. Manajemen Produk & Stok**
- Judul "Manajemen Katalog Produk" + tombol "+ Tambah Produk".
- Tabel 10 per halaman (terbaru di atas): No, Gambar (thumbnail atau placeholder), Nama, Kategori, Stok (badge), Harga (L1), dan aksi **Edit** (oranye), **+Stok** (`#17a2b8`), **−Stok** (`#ffc107`), **Hapus** (`#dc3545`, pakai `confirm()`).
- **Modal produk**:
  - Nama Produk*, Kode Barang/SKU (kosong = dibuat otomatis `PRD-XXXXXXXX`), Brand*, Kategori* (dropdown), **Satuan*** (input teks bebas, default `PCS`), Harga Modal* (`base_price`), Stok Awal*.
  - Panel "Harga Grosir (4 Level)" dengan teks "Atur harga jual berdasarkan kuantitas pengambilan (Level 1: Eceran - Level 4: Grosir Besar)", berisi 4 input harga. Harga yang dikosongkan atau 0 **dihapus** saat update.
  - Upload banyak gambar dengan pratinjau 60px. Edit hanya **menambah** gambar, tidak mengganti yang lama.
- Modal stok: Jumlah* dan Catatan. Pengurangan ditolak kalau jumlahnya melebihi stok ("Jumlah stok yang dikurangi melebihi stok yang tersedia."). Setiap perubahan membuat broadcast `user_id = null`.

**C. POS Kasir (`/admin/pos`)**
- Layout grid `1fr 400px`.
  - Header: input "Cari produk..." (≥2 karakter, filter grid berdasarkan nama/kode/brand) + tombol "🔍 Cari Produk" (modal pencarian).
  - Kiri: grid kartu produk `auto-fill minmax(180px,1fr)` berisi nama, **harga `base_price`**, "Stok: N". Infinite scroll memuat 12 produk tambahan.
  - Kanan: panel "🛒 Keranjang POS".
- **Pilih pelanggan**:
  1. Ketik ≥2 karakter di "Cari nama pelanggan...". Muncul opsi "+ Tambah Walk-in Customer" dan daftar customer (nama – email).
  2. Memilih customer menampilkan "Pelanggan: {nama}".
  3. Memilih walk-in memunculkan input "Nama Walk-in Customer".
- **Tambah item**: klik kartu produk. Klik produk yang sama menambah qty +1. Tidak ada tombol −/+, tidak ada pilihan satuan, dan tidak ada cek stok di sisi klien. Baris keranjang: "{nama} (xN)" dan tombol × merah untuk hapus.
- **Total** = Σ(harga × qty) dari harga kartu (`base_price`).
- **Metode bayar**: dua tombol, "💵 Cash" (default terpilih) dan "💳 Midtrans". Klik "Bayar Sekarang" saat keranjang kosong tidak melakukan apa-apa.
- **Modal Cash**:
  - "Total Pembayaran" (oranye).
  - "Uang Diterima", otomatis terisi sama dengan total.
  - "Kembalian" (readonly), dihitung saat event `change` (setelah blur atau Enter, bukan saat mengetik): `kembalian = diterima − total`. Kalau negatif ditampilkan 0. Tidak ada validasi uang kurang.
  - Tombol "Konfirmasi Pembayaran".
- **Server** (`POST /admin/pos/store`):
  1. Harga per item = harga di level `customer_group` pelanggan (walk-in/`poin` = L1).
  2. Stok dikurangi **tanpa cek** (bisa jadi minus).
  3. Membuat Transaction `POS-{8 karakter acak}` berstatus `processing` (cash) atau `pending` (midtrans), dengan `shipping_address` = nama pelanggan.
  4. Kalau pelanggan terdaftar: `total_spent += total`, lalu naik grup sekali saja dari `poin` (≥1jt → level1, ≥2jt → level2, ≥5jt → level3, ≥10jt → level4).
  5. Toast "Transaksi berhasil!", lalu keranjang dan pelanggan direset.
- Tabel `pos_transactions` ada di database tetapi tidak dipakai.
- Contoh selisih harga: 2 × Busi NGK di layar POS = 2 × 76.000 = **Rp 152.000**, padahal yang tersimpan 2 × 95.000 = **Rp 190.000**.

**D. Stock-In (`/admin/pos/stock-in`)**: form "Tambah Stok Produk" berisi Produk (dropdown 20 produk pertama, "Nama (Stok: N)"), Jumlah, Supplier, Catatan, tombol Simpan. Datanya tersimpan di `stock_movements` dengan type `in`.

**E. Manajemen Pesanan**
- Tabel: No, Pelanggan (POS walk-in tampil "N/A"), Total, badge Status, Tanggal (d/m/Y), tombol "Detail".
- Halaman detail:
  - Kiri: Informasi Pelanggan (Nama, Email, Telepon), Alamat Pengiriman, Detail Produk (gambar, nama, brand, "Qty: N x Rp {harga L1 saat ini}", subtotal dari harga yang tersimpan di transaksi).
  - Kanan: Ringkasan (No. Invoice, Tanggal, Status, Total) dan dropdown **Ubah Status** (Pending / Processing / Shipped / Delivered / Done / Cancelled). Perubahan langsung tersimpan saat dipilih dan mengirim notifikasi ke pelanggan.

**F. Manajemen Pengguna**
- Tabel: No, Nama, Email, Telepon, Level Harga Aktif (badge oranye atau teks "Default"), tombol "Ubah Level".
- Modal: "Ubah Level Harga untuk {nama}", pilih tier*, Masa Berlaku opsional (minimal besok; kosong = permanen).
- Saat disimpan: tier aktif lain dibuat kedaluwarsa, lalu notifikasi promo "Anda mendapatkan level harga {tier} (permanen)" dikirim ke pelanggan.

**G. Level Harga (`/admin/tiers`)**: tabel No, Nama Level, Price Level, Min Pembelian, Pengguna, Edit/Hapus. Modal: Nama*, Price Level* (1 Eceran … 4 Grosir Besar), Min Pembelian*, Deskripsi. Tier yang masih dipakai tidak bisa dihapus.

**H. Kirim Broadcast**
- Form: Judul*, Isi Pesan*, Tipe* (Promo/Sistem/Info), Target* (Semua Customer / Berdasarkan Level Harga / Customer Tertentu).
- Target berdasarkan level menampilkan checkbox tier (label "(0% diskon)"). Target customer tertentu menampilkan pencarian nama/email.
- Tombol "Kirim Broadcast Sekarang", lalu pesan "Notifikasi berhasil dikirim ke N customer."

**I. Laporan Penjualan**: dua kartu tabel, "Pendapatan Bulanan" (12 bulan terakhir, status processing–done) dan "Penjualan per Kategori" (top 5).

---

## 3. Data Dummy Asli dari Seeder (JSON)

Penyesuaian supaya datanya siap pakai:
- Harga dihitung persis dari rumus seeder.
- ID mengikuti urutan insert.
- `barcode` aslinya acak (`'899'.time().$key.rand()`), jadi diganti dengan nilai tetap.

Untuk format tabel `ProductPrice` dan `ProductImage` datar, jalankan:
```js
const product_prices = DB.products.flatMap(p => Object.entries(p.prices).map(([lvl, price]) => ({ product_id: p.id, price_level: +lvl, price })));
const product_images = DB.products.flatMap(p => p.images.map(image_path => ({ product_id: p.id, image_path })));
```

```json
{
  "_catatan": [
    "base_price = Harga Modal = 80% harga eceran. prices: 1=eceran(100%), 2=90%, 3=85%, 4=82%.",
    "Tabel ProductUnit tidak ada; satuan = kolom products.unit, semua 'PCS'.",
    "customer_tier_assignments kosong: UserSeeder mencari tier slug 'retail-b2c' yang tidak ada, jadi semua customer berlevel 1.",
    "Password di bawah adalah password asli seeder (di DB disimpan sebagai hash bcrypt).",
    "File gambar di 'images' tidak ada di repo; URL aslinya /storage/{path}."
  ],
  "categories": [
    {"id":1,"name":"Pengereman","description":"Kategori Pengereman","slug":"pengereman","products_count":6},
    {"id":2,"name":"Kelistrikan","description":"Kategori Kelistrikan","slug":"kelistrikan","products_count":6},
    {"id":3,"name":"Kaki-Kaki & Roda","description":"Kategori Kaki-Kaki & Roda","slug":"kaki-kaki-roda","products_count":6},
    {"id":4,"name":"Mesin & Penggerak","description":"Kategori Mesin & Penggerak","slug":"mesin-penggerak","products_count":6},
    {"id":5,"name":"Oli & Pelumas","description":"Kategori Oli & Pelumas","slug":"oli-pelumas","products_count":6}
  ],
  "product_units": [
    {"unit":"PCS","dipakai_oleh":"semua produk id 1-30"}
  ],
  "customer_tiers": [
    {"id":1,"name":"Level 1 - Eceran","slug":"level-1-eceran","price_level":1,"discount_percent":0,"min_purchase":0,"description":null},
    {"id":2,"name":"Level 2 - Grosir Kecil","slug":"level-2-grosir-kecil","price_level":2,"discount_percent":0,"min_purchase":5000000,"description":null},
    {"id":3,"name":"Level 3 - Grosir Menengah","slug":"level-3-grosir-menengah","price_level":3,"discount_percent":0,"min_purchase":20000000,"description":null},
    {"id":4,"name":"Level 4 - Grosir Besar","slug":"level-4-grosir-besar","price_level":4,"discount_percent":0,"min_purchase":50000000,"description":null}
  ],
  "users": [
    {"id":1,"name":"Admin Partlyfe","email":"admin@partlyfe.com","password":"admin","role":"admin","phone":"081234567890","address":"Jakarta, Indonesia","customer_group":"poin","total_spent":0},
    {"id":2,"name":"user1","email":"user1@email.com","password":"user1","role":"customer","phone":"081234567890","address":"Jl. Rungkut Industri No. 45, Kota Surabaya","customer_group":"poin","total_spent":0},
    {"id":3,"name":"user2","email":"user2@example.com","password":"user2","role":"customer","phone":"082345678901","address":"Jl. Sudirman No. 10, Jakarta Pusat","customer_group":"poin","total_spent":0}
  ],
  "customer_tier_assignments": [],
  "products": [
    {"id":1,"category_id":1,"item_type":"barang","item_code":"PRT-001","barcode":"8990000000001","name":"Kampas Rem Depan Vario / Beat FI","brand":"AHM","unit":"PCS","base_price":44000,"current_stock":50,"cashback_percent":0,"prices":{"1":55000,"2":49500,"3":46750,"4":45100},"images":["products/img/kampas-depan-1.png","products/img/kampas-depan-detail.png"]},
    {"id":2,"category_id":1,"item_type":"barang","item_code":"PRT-002","barcode":"8990000000002","name":"Kampas Rem Belakang NMAX / Aerox","brand":"YGP","unit":"PCS","base_price":52000,"current_stock":45,"cashback_percent":0,"prices":{"1":65000,"2":58500,"3":55250,"4":53300},"images":["products/img/kampas-nmax.png"]},
    {"id":3,"category_id":1,"item_type":"barang","item_code":"PRT-003","barcode":"8990000000003","name":"Piringan Cakram Depan Supra X 125","brand":"Aspira","unit":"PCS","base_price":100000,"current_stock":20,"cashback_percent":0,"prices":{"1":125000,"2":112500,"3":106250,"4":102500},"images":["products/img/cakram-supra.png"]},
    {"id":4,"category_id":1,"item_type":"barang","item_code":"PRT-004","barcode":"8990000000004","name":"Master Rem Atas Kanan Universal","brand":"Choho","unit":"PCS","base_price":140000,"current_stock":15,"cashback_percent":0,"prices":{"1":175000,"2":157500,"3":148750,"4":143500},"images":["products/img/master-rem.png"]},
    {"id":5,"category_id":1,"item_type":"barang","item_code":"PRT-005","barcode":"8990000000005","name":"Kabel Rem Belakang Mio Karbu","brand":"Federal","unit":"PCS","base_price":28000,"current_stock":30,"cashback_percent":0,"prices":{"1":35000,"2":31500,"3":29750,"4":28700},"images":["products/img/kabel-rem.png"]},
    {"id":6,"category_id":1,"item_type":"barang","item_code":"PRT-006","barcode":"8990000000006","name":"Caliper Rem Bawah Belakang Satria FU","brand":"MPM","unit":"PCS","base_price":168000,"current_stock":8,"cashback_percent":0,"prices":{"1":210000,"2":189000,"3":178500,"4":172200},"images":["products/img/caliper.png"]},
    {"id":7,"category_id":2,"item_type":"barang","item_code":"PRT-007","barcode":"8990000000007","name":"Busi Racing Iridium Power IU22","brand":"NGK","unit":"PCS","base_price":76000,"current_stock":120,"cashback_percent":0,"prices":{"1":95000,"2":85500,"3":80750,"4":77900},"images":["products/img/busi-iridium.png","products/img/busi-box.png"]},
    {"id":8,"category_id":2,"item_type":"barang","item_code":"PRT-008","barcode":"8990000000008","name":"Aki Kering MF GTZ5S Super Power","brand":"Max1","unit":"PCS","base_price":148000,"current_stock":25,"cashback_percent":0,"prices":{"1":185000,"2":166500,"3":157250,"4":151700},"images":["products/img/aki-gtz5s.png"]},
    {"id":9,"category_id":2,"item_type":"barang","item_code":"PRT-009","barcode":"8990000000009","name":"Kiprok Regulator Karisma / Supra X","brand":"Federal","unit":"PCS","base_price":60000,"current_stock":18,"cashback_percent":0,"prices":{"1":75000,"2":67500,"3":63750,"4":61500},"images":["products/img/kiprok.png"]},
    {"id":10,"category_id":2,"item_type":"barang","item_code":"PRT-010","barcode":"8990000000010","name":"CDI Racing Unlimiter Jupiter Z Lama","brand":"Tenshi","unit":"PCS","base_price":128000,"current_stock":12,"cashback_percent":0,"prices":{"1":160000,"2":144000,"3":136000,"4":131200},"images":["products/img/cdi-racing.png"]},
    {"id":11,"category_id":2,"item_type":"barang","item_code":"PRT-011","barcode":"8990000000011","name":"Bohlam Lampu Depan LED H4 Kaki 3","brand":"Thai","unit":"PCS","base_price":38400,"current_stock":60,"cashback_percent":0,"prices":{"1":48000,"2":43200,"3":40800,"4":39360},"images":["products/img/led-h4.png"]},
    {"id":12,"category_id":2,"item_type":"barang","item_code":"PRT-012","barcode":"8990000000012","name":"Spool Stator Assy Grand / Prima","brand":"NPP","unit":"PCS","base_price":92000,"current_stock":10,"cashback_percent":0,"prices":{"1":115000,"2":103500,"3":97750,"4":94300},"images":["products/img/spool.png"]},
    {"id":13,"category_id":3,"item_type":"barang","item_code":"PRT-013","barcode":"8990000000013","name":"Ban Luar Tubeless Green Devil 90/80-14","brand":"Maxxis","unit":"PCS","base_price":204000,"current_stock":40,"cashback_percent":0,"prices":{"1":255000,"2":229500,"3":216750,"4":209100},"images":["products/img/maxxis-devil.png","products/img/maxxis-tread.png"]},
    {"id":14,"category_id":3,"item_type":"barang","item_code":"PRT-014","barcode":"8990000000014","name":"Ban Luar NR73 Tube Type 80/90-17","brand":"IRC","unit":"PCS","base_price":142400,"current_stock":35,"cashback_percent":0,"prices":{"1":178000,"2":160200,"3":151300,"4":145960},"images":["products/img/rc-nr73.png"]},
    {"id":15,"category_id":3,"item_type":"barang","item_code":"PRT-015","barcode":"8990000000015","name":"Shockbreaker Belakang Pro-Series 330mm","brand":"Aspira","unit":"PCS","base_price":236000,"current_stock":14,"cashback_percent":0,"prices":{"1":295000,"2":265500,"3":250750,"4":241900},"images":["products/img/shock-aspira.png","products/img/shock-box.png"]},
    {"id":16,"category_id":3,"item_type":"barang","item_code":"PRT-016","barcode":"8990000000016","name":"V-Belt Kit + Roller Vario 110 Karbu","brand":"MPM","unit":"PCS","base_price":108000,"current_stock":22,"cashback_percent":0,"prices":{"1":135000,"2":121500,"3":114750,"4":110700},"images":["products/img/vbelt-vario.png"]},
    {"id":17,"category_id":3,"item_type":"barang","item_code":"PRT-017","barcode":"8990000000017","name":"Gear Set Paket Rantai Supra Fit New","brand":"Choho","unit":"PCS","base_price":132000,"current_stock":16,"cashback_percent":0,"prices":{"1":165000,"2":148500,"3":140250,"4":135300},"images":["products/img/gearset.png"]},
    {"id":18,"category_id":3,"item_type":"barang","item_code":"PRT-018","barcode":"8990000000018","name":"Laher Bearing Roda Depan 6301 ZZ","brand":"NPP","unit":"PCS","base_price":14400,"current_stock":80,"cashback_percent":0,"prices":{"1":18000,"2":16200,"3":15300,"4":14760},"images":["products/img/bearing.png"]},
    {"id":19,"category_id":4,"item_type":"barang","item_code":"PRT-019","barcode":"8990000000019","name":"Piston Kit Bore Up diameter 58mm","brand":"NPP","unit":"PCS","base_price":124000,"current_stock":25,"cashback_percent":0,"prices":{"1":155000,"2":139500,"3":131750,"4":127100},"images":["products/img/piston-kit.png","products/img/piston-detail.png"]},
    {"id":20,"category_id":4,"item_type":"barang","item_code":"PRT-020","barcode":"8990000000020","name":"Karburator Assy Black PE 28 Racing","brand":"Mikuni","unit":"PCS","base_price":272000,"current_stock":9,"cashback_percent":0,"prices":{"1":340000,"2":306000,"3":289000,"4":278800},"images":["products/img/karbu-pe28.png"]},
    {"id":21,"category_id":4,"item_type":"barang","item_code":"PRT-021","barcode":"8990000000021","name":"Blok Cylinder Set Standar Grand","brand":"Federal","unit":"PCS","base_price":212000,"current_stock":6,"cashback_percent":0,"prices":{"1":265000,"2":238500,"3":225250,"4":217300},"images":["products/img/blok-silinder.png"]},
    {"id":22,"category_id":4,"item_type":"barang","item_code":"PRT-022","barcode":"8990000000022","name":"Noken As Camshaft Racing Mio Karbu","brand":"Tenshi","unit":"PCS","base_price":100000,"current_stock":15,"cashback_percent":0,"prices":{"1":125000,"2":112500,"3":106250,"4":102500},"images":["products/img/noken-as.png"]},
    {"id":23,"category_id":4,"item_type":"barang","item_code":"PRT-023","barcode":"8990000000023","name":"Kampas Kopling Ganda Matic Beat FI","brand":"AHM","unit":"PCS","base_price":92000,"current_stock":0,"cashback_percent":0,"prices":{"1":115000,"2":103500,"3":97750,"4":94300},"images":["products/img/kopling-ganda.png"]},
    {"id":24,"category_id":4,"item_type":"barang","item_code":"PRT-024","barcode":"8990000000024","name":"Stang Seher Connecting Rod Legenda","brand":"NPP","unit":"PCS","base_price":84000,"current_stock":0,"cashback_percent":0,"prices":{"1":105000,"2":94500,"3":89250,"4":86100},"images":["products/img/stang-seher.png"]},
    {"id":25,"category_id":5,"item_type":"barang","item_code":"PRT-025","barcode":"8990000000025","name":"Oli Mesin MPX 2 Matic 10W-30 0.8L","brand":"AHM","unit":"PCS","base_price":43200,"current_stock":150,"cashback_percent":0,"prices":{"1":54000,"2":48600,"3":45900,"4":44280},"images":["products/img/oli-mpx2.png"]},
    {"id":26,"category_id":5,"item_type":"barang","item_code":"PRT-026","barcode":"8990000000026","name":"Oli Mesin Yamalube Super Matic 1L","brand":"YGP","unit":"PCS","base_price":56000,"current_stock":90,"cashback_percent":0,"prices":{"1":70000,"2":63000,"3":59500,"4":57400},"images":["products/img/oli-yamalube.png"]},
    {"id":27,"category_id":5,"item_type":"barang","item_code":"PRT-027","barcode":"8990000000027","name":"Oli Samping 2T Premium Low Smoke 1L","brand":"Thai","unit":"PCS","base_price":36000,"current_stock":40,"cashback_percent":0,"prices":{"1":45000,"2":40500,"3":38250,"4":36900},"images":["products/img/oli-2t.png"]},
    {"id":28,"category_id":5,"item_type":"barang","item_code":"PRT-028","barcode":"8990000000028","name":"Minyak Rem DOT 4 Super Brake Fluid","brand":"MPM","unit":"PCS","base_price":22400,"current_stock":55,"cashback_percent":0,"prices":{"1":28000,"2":25200,"3":23800,"4":22960},"images":["products/img/minyak-rem.png"]},
    {"id":29,"category_id":5,"item_type":"barang","item_code":"PRT-029","barcode":"8990000000029","name":"Oli Gardan Gear Oil Matic 120ml","brand":"Aspira","unit":"PCS","base_price":13600,"current_stock":110,"cashback_percent":0,"prices":{"1":17000,"2":15300,"3":14450,"4":13940},"images":["products/img/oli-gardan.png"]},
    {"id":30,"category_id":5,"item_type":"barang","item_code":"PRT-030","barcode":"8990000000030","name":"Air Radiator Coolant Super Long Life","brand":"AHM","unit":"PCS","base_price":19200,"current_stock":0,"cashback_percent":0,"prices":{"1":24000,"2":21600,"3":20400,"4":19680},"images":["products/img/coolant.png"]}
  ],
  "_opsional_template_transactions": {
    "_catatan": "Dari code_template/seeders/TransactionSeeder.php (TIDAK dipanggil DatabaseSeeder aktif). Aslinya produk dipilih acak dan user = User::first() (admin). product_id, user_id, nomor invoice, dan tanggal di bawah adalah pilihan contoh.",
    "transactions": [
      {"id":1,"invoice_number":"INV-PL-17640000001","user_id":2,"total_amount":190000,"status":"delivered","payment_method":"Bank Transfer (BCA)","shipping_address":"Jl. Boulevard CitraRaya Blok A No. 1, Surabaya","created_at":"2026-11-20T10:15:00+07:00"},
      {"id":2,"invoice_number":"INV-PL-17640000002","user_id":2,"total_amount":309000,"status":"processing","payment_method":"E-Wallet (GoPay)","shipping_address":"Gedung Universitas Ciputra, Ruang Lab AI","created_at":"2026-11-24T14:30:00+07:00"}
    ],
    "transaction_details": [
      {"transaction_id":1,"product_id":7,"qty":2,"price":95000},
      {"transaction_id":2,"product_id":25,"qty":1,"price":54000},
      {"transaction_id":2,"product_id":13,"qty":1,"price":255000}
    ]
  }
}
```

**Format nomor invoice** (berguna untuk simulasi di demo): checkout online `TRX-{unix_time}-{user_id}`, POS `POS-{8 huruf/angka kapital acak}`, template `INV-PL-{time}{n}`.

---

## 4. Aset Gambar

**Isi `public/`:** hanya `favicon.ico`, `robots.txt`, `index.php`, `.htaccess`, dan `build/` (CSS/JS Vite serta font Instrument Sans yang tidak dipakai). Tidak ada logo, banner, maupun foto produk.

| Aset | Sumber asli | Status |
|---|---|---|
| Logo | Teks `PART<span>L</span>YFE` (Bebas Neue, L oranye) | Tidak ada file gambar |
| Favicon | `public/favicon.ico` | Ada (ikon bawaan Laravel) |
| Banner hero | SVG inline di `home.blade.php` (di bawah) | Inline |
| Banner promo | Kotak CSS oranye + teks "SALE" | Inline |
| Ikon kategori | 4 SVG inline (di bawah) | Inline |
| Foto produk | 35 path `products/img/*.png` (field `images` di JSON), URL `/storage/products/img/<file>.png` | **Semua tidak ada** |
| Upload admin | `storage/app/public/products/<hash>.<ext>` | Kosong |

**Fallback per halaman** (tampil kalau produk tidak punya baris gambar). Untuk demo, pakai `onerror` agar path yang rusak juga diganti placeholder.

```html
<!-- Home (unggulan), 120px, class produk-img-placeholder opacity .18 -->
<svg width="120" height="120" viewBox="0 0 100 100" style="opacity:.18"><circle cx="50" cy="50" r="42" fill="none" stroke="#E8E3D6" stroke-width="1.5"/><circle cx="50" cy="50" r="28" fill="none" stroke="#E8E3D6" stroke-width="1"/><circle cx="50" cy="50" r="10" fill="#E8E3D6"/></svg>

<!-- Katalog, 80px, opacity .15; juga dropdown search (24px) dan AI chat (30px, rx=8) -->
<svg width="80" height="80" viewBox="0 0 100 100" style="opacity:.15"><rect x="15" y="15" width="70" height="70" rx="4" fill="none" stroke="#F2EFE6" stroke-width="1.5" opacity=".3"/></svg>

<!-- Detail (gambar utama), 160px, opacity .15 (.25 saat hover) -->
<svg width="160" height="160" viewBox="0 0 100 100" style="opacity:.15"><circle cx="50" cy="50" r="42" fill="none" stroke="#F2EFE6" stroke-width="1.2"/><circle cx="50" cy="50" r="25" fill="none" stroke="#F2EFE6" stroke-width=".8" stroke-dasharray="2,2"/><path d="M50 10 L50 90 M10 50 L90 50" stroke="#F2EFE6" stroke-width=".5" opacity=".3"/></svg>
<!-- Detail (thumbnail): 1 lingkaran r=35 + 3 kotak rect 20,20,60,60 opacity .4 -->

<!-- Keranjang, wishlist, rekomendasi, admin: ikon "image" -->
<svg width="40" height="40" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.2" opacity=".4"><rect x="3" y="3" width="18" height="18" rx="2"/><circle cx="8.5" cy="8.5" r="1.5"/><path d="M21 15l-5-5L5 21"/></svg>
```

**Ikon kategori di Home** (`fill: #F2EFE6`, 140px, opacity .12). Dipilih berdasarkan kata dalam nama kategori:
```html
<!-- nama mengandung "mesin" -->
<svg viewBox="0 0 100 100"><rect x="20" y="25" width="60" height="50" rx="4" fill="currentColor"/><rect x="35" y="10" width="30" height="20" rx="3" fill="currentColor"/><rect x="10" y="40" width="15" height="20" rx="2" fill="currentColor"/><rect x="75" y="40" width="15" height="20" rx="2" fill="currentColor"/><circle cx="50" cy="80" r="12" fill="currentColor"/></svg>
<!-- mengandung "rem" / "pengereman" -->
<svg viewBox="0 0 100 100"><circle cx="50" cy="50" r="40" fill="currentColor"/><circle cx="50" cy="50" r="25" fill="#141410"/><circle cx="50" cy="50" r="10" fill="currentColor"/><rect x="46" y="5" width="8" height="20" rx="2" fill="currentColor"/><rect x="46" y="75" width="8" height="20" rx="2" fill="currentColor"/><rect x="5" y="46" width="20" height="8" rx="2" fill="currentColor"/><rect x="75" y="46" width="20" height="8" rx="2" fill="currentColor"/></svg>
<!-- mengandung "listrik" -->
<svg viewBox="0 0 100 100"><polygon points="55,5 30,55 50,55 45,95 70,45 50,45" fill="currentColor"/></svg>
<!-- lainnya (Kaki-Kaki & Roda, Oli & Pelumas) -->
<svg viewBox="0 0 100 100"><rect x="10" y="30" width="80" height="40" rx="6" fill="currentColor"/><rect x="20" y="20" width="60" height="15" rx="3" fill="currentColor"/><circle cx="25" cy="78" r="8" fill="currentColor"/><circle cx="75" cy="78" r="8" fill="currentColor"/></svg>
```

**Ilustrasi hero** (SVG asli lengkap):
```html
<svg viewBox="0 0 800 900" xmlns="http://www.w3.org/2000/svg" style="width:100%;height:100%;position:absolute;inset:0;opacity:.7">
  <defs><radialGradient id="glow" cx="50%" cy="50%" r="50%"><stop offset="0%" stop-color="#E8521A" stop-opacity="0.15"/><stop offset="100%" stop-color="#0A0A08" stop-opacity="0"/></radialGradient></defs>
  <rect width="800" height="900" fill="#0F0F0D"/>
  <ellipse cx="400" cy="450" rx="350" ry="350" fill="url(#glow)"/>
  <g transform="translate(160, 180)" opacity="0.6">
    <rect x="140" y="80" width="200" height="260" rx="8" fill="none" stroke="#E8521A" stroke-width="1.5"/>
    <rect x="160" y="100" width="160" height="40" rx="4" fill="none" stroke="#C8C4B8" stroke-width="1" opacity="0.5"/>
    <rect x="160" y="150" width="160" height="40" rx="4" fill="none" stroke="#C8C4B8" stroke-width="1" opacity="0.5"/>
    <rect x="160" y="200" width="160" height="40" rx="4" fill="none" stroke="#C8C4B8" stroke-width="1" opacity="0.5"/>
    <rect x="160" y="250" width="160" height="40" rx="4" fill="none" stroke="#C8C4B8" stroke-width="1" opacity="0.5"/>
    <rect x="120" y="40" width="240" height="50" rx="6" fill="none" stroke="#E8521A" stroke-width="2"/>
    <circle cx="180" cy="65" r="12" fill="none" stroke="#888880" stroke-width="1"/>
    <circle cx="240" cy="65" r="12" fill="none" stroke="#888880" stroke-width="1"/>
    <circle cx="300" cy="65" r="12" fill="none" stroke="#888880" stroke-width="1"/>
    <path d="M340 200 Q420 200 420 280 Q420 360 340 360" fill="none" stroke="#E8521A" stroke-width="2" stroke-dasharray="6,4" opacity="0.7"/>
    <rect x="140" y="340" width="200" height="60" rx="6" fill="none" stroke="#888880" stroke-width="1" opacity="0.5"/>
    <path d="M140 160 Q60 160 60 240 Q60 300 140 300" fill="none" stroke="#C8C4B8" stroke-width="1.5" opacity="0.4"/>
    <circle cx="240" cy="460" r="70" fill="none" stroke="#E8521A" stroke-width="2" opacity="0.5"/>
    <circle cx="240" cy="460" r="40" fill="none" stroke="#888880" stroke-width="1" opacity="0.4"/>
    <line x1="240" y1="390" x2="240" y2="340" stroke="#C8C4B8" stroke-width="1.5" opacity="0.5"/>
    <circle cx="148" cy="88" r="5" fill="#1A1A16" stroke="#888880" stroke-width="1" opacity="0.6"/>
    <circle cx="332" cy="88" r="5" fill="#1A1A16" stroke="#888880" stroke-width="1" opacity="0.6"/>
    <circle cx="148" cy="332" r="5" fill="#1A1A16" stroke="#888880" stroke-width="1" opacity="0.6"/>
    <circle cx="332" cy="332" r="5" fill="#1A1A16" stroke="#888880" stroke-width="1" opacity="0.6"/>
  </g>
</svg>
```

---

## 5. Perilaku asli yang bermasalah (tentukan: ditiru atau diperbaiki di demo)

1. **Link kategori 404.** `/kategori/{slug}` mencari nama dengan `slug` yang tanda `-`-nya diganti spasi. "Kaki-Kaki & Roda", "Mesin & Penggerak", dan "Oli & Pelumas" tidak pernah cocok, jadi **2 dari 4 kartu kategori di Beranda membuka 404**. Di demo, filter pakai `category_id`.
2. **Filter harga katalog membaca `base_price`** (harga modal), padahal kartu menampilkan harga level. Akibatnya rentang slider (13.600–272.000) tidak sesuai dengan harga yang terlihat.
3. **POS menampilkan `base_price` tetapi menagih harga level** (lihat contoh Rp 152.000 vs Rp 190.000 di §2.4.C). Stok juga bisa menjadi minus.
4. **Stepper keranjang**: nilai qty pada tombol tertanam saat render, jadi klik berulang tidak menambah lagi. Respons server menimpa harga satuan dengan subtotal, dan ringkasan total tidak ikut ter-update.
5. **Error keranjang tampil sebagai "Terjadi kesalahan jaringan."** Untuk stok habis atau melebihi stok, server mengembalikan redirect, bukan JSON. Di detail produk dengan stok 0, stepper malah mengizinkan sampai 24 (`parseInt('0') || 24`).
6. **"Beli Sekarang" menghapus isi keranjang lain.**
7. **Tier pelanggan seeder tidak pernah terpasang** (slug `retail-b2c` tidak ada), jadi harga grosir tidak terlihat tanpa assign manual dari admin. Untuk demo, bisa dipasang Level 2 ke `user2`; itu tambahan sendiri, bukan data seeder.
8. **Halaman error**: `/dashboard` (variabel `$tierPriceLevel` tidak dikirim) dan `/admin/tiers` (memanggil `currentPage()` pada Collection). Di halaman tiers, kolom "Price Level" juga menampilkan nama tier, bukan angkanya.
9. **Data palsu atau nol**: rating "4.8" di katalog; rating/ulasan/terjual = 0 dan spesifikasi "-" di detail; "Total Transaksi" di profil selalu 0; "Total Belanja" pelanggan baru di dashboard selalu "Rp 0".
10. **Notifikasi dari admin** untuk produk baru dan perubahan stok disimpan dengan `user_id = null`, jadi tidak pernah muncul di sisi pelanggan.

**Catatan keamanan:** route `/sync/receive` dan `/sync/pull` di [routes/web.php](routes/web.php) berada di luar grup `auth`. Keduanya hanya dijaga token yang ditulis langsung di kode (`PartLyfeSyncSecure999`, [CustomerController.php:909](app/Http/Controllers/CustomerController.php#L909)) dan menerima nama tabel apa pun dari request. Siapa pun yang pernah melihat kode ini bisa membaca tabel `users` (termasuk hash password) atau menulis ke tabel mana saja di server Hugging Face Spaces. Kalau repo ini publik, sebaiknya token diganti dan route itu dikunci.
