# ELİF OS — Kurulum

## Ön Koşullar

- Python 3.11+ (bu proje Python 3.14 ile geliştirilip test edildi)
- Node.js 20+ ve npm
- Git

## 1. Backend

```powershell
cd backend

# Sanal ortam oluştur ve etkinleştir
python -m venv .venv
.\.venv\Scripts\Activate.ps1

# Bağımlılıkları kur
pip install -r requirements.txt

# Ortam değişkenlerini ayarla
copy .env.example .env
```

`.env` dosyasını açın ve en azından `SECRET_KEY` alanını doldurun:

```powershell
python -c "import secrets; print(secrets.token_urlsafe(48))"
```

Çıktıyı `.env` içindeki `SECRET_KEY=` satırına yapıştırın.

### Veritabanı migration'larını çalıştırın

```powershell
python -m alembic upgrade head
```

Bu, `backend/elif_os.db` adında bir SQLite dosyası oluşturur ve tüm tabloları
kurar.

### Backend'i başlatın

```powershell
python -m uvicorn app.main:app --reload --port 8000
```

`http://localhost:8000/api/health` adresine gidip `{"status": "ok"}` yanıtını
görmelisiniz.

### Backend testlerini çalıştırın

```powershell
python -m pytest tests/ -v
```

## 2. Frontend

Yeni bir terminalde:

```powershell
cd frontend
npm install
npm run dev
```

`http://localhost:5173` adresini tarayıcıda açın. Vite, `/api/*` isteklerini
otomatik olarak `http://127.0.0.1:8000`'e yönlendirir (bkz. `vite.config.ts`).

### Frontend testlerini çalıştırın

```powershell
npm run test
```

### Üretim derlemesi

```powershell
npm run build
```

## 3. İsteğe Bağlı: Anthropic (Claude) API

Brain Dump, AI Asistan, WHAT NOW açıklamaları ve Haftalık Değerlendirme
gözlemleri için gereklidir. Olmadan da uygulama tam olarak çalışır — bu
özellikler arayüzde "AI yapılandırılmadı" mesajı gösterir.

1. https://console.anthropic.com adresinden bir API anahtarı alın.
2. `backend/.env` içine ekleyin:
   ```
   ANTHROPIC_API_KEY=sk-ant-...
   ANTHROPIC_MODEL=claude-sonnet-5
   ```
3. Backend'i yeniden başlatın.

## 4. İsteğe Bağlı: Google Calendar Entegrasyonu

1. https://console.cloud.google.com adresinde bir proje oluşturun.
2. "APIs & Services → Credentials" bölümünden **OAuth 2.0 Client ID**
   (Application type: **Web application**) oluşturun.
3. **Authorized redirect URIs** alanına şunu ekleyin:
   ```
   http://localhost:8000/api/calendar/oauth/callback
   ```
4. "APIs & Services → Library" bölümünden **Google Calendar API**'yi
   etkinleştirin.
5. Client ID ve Client Secret'ı `backend/.env` içine ekleyin:
   ```
   GOOGLE_CLIENT_ID=...
   GOOGLE_CLIENT_SECRET=...
   ```
6. Backend'i yeniden başlatın. Ayarlar sayfasında "Google Calendar'ı Bağla"
   butonu görünecektir.

## Sorun Giderme

**`ModuleNotFoundError` backend'i başlatırken**
Sanal ortamın etkin olduğundan ve `pip install -r requirements.txt`'nin
başarıyla tamamlandığından emin olun.

**Frontend `/api` isteklerinde bağlantı hatası veriyor**
Backend'in `127.0.0.1:8000` üzerinde çalıştığından emin olun. `vite.config.ts`
içindeki proxy hedefi budur.

**"AI yapılandırılmadı" hatası her yerde görünüyor**
`ANTHROPIC_API_KEY` `.env` dosyasında boş ya da tanımsız. Doldurup backend'i
yeniden başlatın.

**Alembic "target database is not up to date" hatası**
`python -m alembic upgrade head` komutunu tekrar çalıştırın.

**bcrypt / passlib hatası**
Bu proje `passlib` yerine doğrudan `bcrypt` kütüphanesini kullanır (yeni
bcrypt sürümleriyle passlib'in uyumsuzluğu nedeniyle). `pip install -r
requirements.txt` güncel kalınca sorun oluşmamalıdır.
