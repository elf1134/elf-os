# ELİF OS

> "Sana sadece ne yapman gerektiğini söylemez. Durumunu anlar ve şu an ne
> yapmanın en mantıklı olduğunu söyler."

ELİF OS, hedeflerinizi görevlere, günlük planınıza ve "şimdi ne yapmalıyım"
sorusuna dönüştüren kişisel bir yapay zeka işletim sistemidir.

```
HEDEFLER → PROJELER → KİLOMETRE TAŞLARI → GÖREVLER → GÜNLÜK PLAN → SONRAKİ ADIM
```

Bu bir prototip değildir — gerçek bir veritabanı, gerçek kimlik doğrulama ve
gerçek (isteğe bağlı) Claude API entegrasyonu ile çalışan tam işlevli bir
uygulamadır.

## Özellikler

- **WHAT NOW ("Şimdi Ne Yapmalıyım?")** — kural tabanlı bir skorlama
  algoritmasının, mevcut zamanınıza, önceliklere, son tarihlere ve enerjinize
  göre seçtiği tek bir sonraki adım.
- **Brain Dump** — serbest metni AI ile yapılandırılmış görev taslaklarına
  ayırır; siz onaylamadan hiçbir şey kaydedilmez.
- **Hedef → Proje → Kilometre Taşı → Görev** hiyerarşisi, otomatik ilerleme
  hesaplamasıyla.
- **Akademik Modül** — dersler, konular, ödev/sınavlar, çalışma oturumları ve
  "Ne Çalışmalıyım?" önerisi.
- **Deadline Intelligence** — her son tarih için gerçekçi risk seviyesi
  (kalan iş / gerçekçi kullanılabilir zaman oranına dayalı, açıkça "tahmin"
  olarak etiketlenir).
- **Günlük Planlayıcı** — sabit takvim etkinliklerini, öncelikleri, enerjiyi
  ve molaları dikkate alan, sürükle-bırak destekli bir zamanlama motoru.
- **Odak Modu** — gerçek zamanlı geri sayım sayacı; harcanan süre gelecekteki
  planlamayı iyileştirmek için kaydedilir.
- **Haftalık Değerlendirme** — gerçek verilerden türetilmiş istatistikler ve
  gözlemler.
- **AI Asistan** — araç çağırma (tool calling) ile görevlerinizi
  yönetebilen bir sohbet arayüzü; yıkıcı işlemler için onay ister.
- **Google Calendar entegrasyonu** (OAuth 2.0, salt okunur) — kimlik bilgisi
  eklenene kadar arayüzde "yapılandırılmadı" olarak görünür.
- Açık/koyu tema, mobil uyumlu arayüz, bildirimler, enerji takibi.

## Teknoloji Yığını

| Katman | Teknoloji |
|---|---|
| Frontend | React 19 + TypeScript + Vite + Tailwind CSS v4 |
| Frontend durum/veri | Zustand (istemci durumu), TanStack Query (sunucu durumu) |
| Backend | Python + FastAPI + Pydantic v2 |
| Veritabanı | SQLite + SQLAlchemy ORM + Alembic migration'ları |
| AI | Anthropic Claude API (yapılandırılmış çıktı + tool calling) |
| Takvim | Google Calendar API (OAuth 2.0) |
| Kimlik doğrulama | JWT (access + refresh token), bcrypt şifre hash'leme |

Mimari kararların gerekçesi için bkz. [ARCHITECTURE.md](ARCHITECTURE.md).

## Hızlı Başlangıç

Tam adımlar için [SETUP.md](SETUP.md) dosyasına bakın. Özet:

```powershell
# Backend
cd backend
python -m venv .venv
.\.venv\Scripts\pip install -r requirements.txt
copy .env.example .env   # sonra .env içini doldurun
.\.venv\Scripts\python -m alembic upgrade head
.\.venv\Scripts\python -m uvicorn app.main:app --reload --port 8000

# Frontend (ayrı bir terminalde)
cd frontend
npm install
npm run dev
```

Tarayıcıda `http://localhost:5173` adresini açın.

## Gerekli Kimlik Bilgileri

| Değişken | Zorunlu mu? | Olmadan ne olur |
|---|---|---|
| `SECRET_KEY` | Evet | JWT imzalama için gerekli; rastgele üretin |
| `ANTHROPIC_API_KEY` | Hayır | AI özellikleri (Brain Dump, Asistan) arayüzde "yapılandırılmadı" gösterir |
| `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` | Hayır | Takvim entegrasyonu arayüzde "yapılandırılmadı" gösterir |

## Belgeler

- [ARCHITECTURE.md](ARCHITECTURE.md) — sistem mimarisi ve tasarım kararları
- [SETUP.md](SETUP.md) — adım adım kurulum
- [SECURITY.md](SECURITY.md) — güvenlik denetimi ve bilinen sınırlamalar
- [ROADMAP.md](ROADMAP.md) — tamamlanan/eksik özellik takibi
- [LEARNING_GUIDE.md](LEARNING_GUIDE.md) — kod tabanını öğrenmek isteyenler için
- [RELEASE_NOTES.md](RELEASE_NOTES.md) — sürüm notları
