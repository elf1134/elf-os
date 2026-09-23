# ELİF OS — Mimari

## Genel Bakış

ELİF OS iki ayrı uygulamadan oluşur ve HTTP/JSON üzerinden konuşur:

```
frontend/   React + TypeScript + Vite + Tailwind  (tarayıcıda çalışır)
backend/    Python + FastAPI + SQLAlchemy + SQLite  (localhost:8000'de çalışır)
```

Frontend, backend'e `/api/*` uç noktaları üzerinden istek atar. Kimlik doğrulama
JWT ile yapılır (access token, `Authorization: Bearer <token>` header'ında taşınır).

Yapay zeka (Anthropic Claude API) **sadece backend içinde** çağrılır. Frontend hiçbir
zaman AI API anahtarına erişemez. AI, veritabanını asla doğrudan değiştiremez —
her AI aracı (tool) aslında doğrulanmış bir backend servis fonksiyonunu çağırır.

## Backend Katmanları

```
backend/app/
  main.py           FastAPI uygulaması, router'ların bağlandığı yer
  core/
    config.py       Ortam değişkenleri (Pydantic Settings)
    database.py     SQLAlchemy engine + session
    security.py     Şifre hashleme, JWT üretme/doğrulama
    deps.py         FastAPI dependency'leri (get_db, get_current_user)
  models/           SQLAlchemy ORM modelleri (veritabanı tabloları)
  schemas/          Pydantic şemaları (API girdi/çıktı doğrulama)
  api/              Router'lar — her biri bir kaynak grubuna karşılık gelir
  services/         İş mantığı: zamanlama motoru, deadline analizi, WHAT NOW skorlama
  ai/               Claude istemcisi, tool tanımları, brain dump ayrıştırıcı, prompt'lar
```

**Neden bu ayrım?** Router'lar (api/) sadece HTTP isteğini alır, doğrular, servise
devreder ve yanıtı döner. Gerçek mantık services/ içindedir — böylece hem testler
hem de AI tool'ları aynı servis fonksiyonlarını çağırabilir, kod tekrarlanmaz.

## Veri Akışı (Örnek: Brain Dump)

```
Kullanıcı serbest metin yazar
  → POST /api/ai/brain-dump  (sadece metni AI'a gönderir, HENÜZ KAYDETMEZ)
  → ai/brain_dump.py: Claude'a structured output isteği (JSON şema ile)
  → Backend, AI çıktısını Pydantic ile doğrular (uydurma alan varsa reddeder)
  → Frontend'e "taslak görev listesi" döner
  → Kullanıcı düzenler, onaylar
  → POST /api/tasks/bulk  (gerçek kayıt burada olur, doğrulanmış servis fonksiyonuyla)
```

AI hiçbir zaman `INSERT`/`UPDATE` çalıştırmaz; sadece yapılandırılmış öneri üretir.

## Zamanlama Motoru (Scheduling Engine)

`services/scheduling.py` saf Python ile çalışan kural tabanlı bir algoritmadır
(LLM'e bağımlı değildir). Girdi: bugünün görevleri, sabit takvim etkinlikleri,
kullanılabilir zaman aralıkları, enerji seviyesi. Çıktı: zaman bloklarının listesi
+ sığmayan görevlerin nedeni. LLM sadece sonucu doğal dille açıklamak için kullanılır
(`ai/explain.py`), kararı vermez.

## WHAT NOW Algoritması

`services/what_now.py`: her açık görev için bir skor hesaplar
(öncelik ağırlığı + son tarihe yakınlık + kalan süreye sığma + enerji uyumu +
bağımlılıkların tamamlanmış olması). En yüksek skorlu görev seçilir. Sonuç, isteğe
bağlı olarak Claude ile tek cümlelik gerekçeye dönüştürülür. Algoritma LLM'siz de
çalışır (LLM devre dışıysa kural tabanlı gerekçe metni kullanılır).

## Kimlik Doğrulama

- Kayıt: e-posta + şifre → bcrypt ile hashlenir (`passlib`).
- Giriş: doğrulama sonrası kısa ömürlü JWT access token + uzun ömürlü refresh token.
- Korumalı uç noktalar `get_current_user` dependency'siyle çalışır.
- Frontend, token'ı `localStorage`'da tutar ve her istekte header'a ekler.

## Google Calendar Entegrasyonu

OAuth 2.0 Authorization Code akışı kullanılır. `backend/app/api/calendar.py`
kullanıcıyı Google'ın izin ekranına yönlendirir, dönen `code`'u access/refresh
token'a çevirip veritabanında (kullanıcıya bağlı, şifrelenmeden **saklanmaz**
— bkz. SECURITY.md) saklar. `GOOGLE_CLIENT_ID`/`GOOGLE_CLIENT_SECRET` ortam
değişkenleri tanımlı değilse bu özellik arayüzde "bağlı değil" olarak görünür
ve devre dışı kalır; sahte veriyle doldurulmaz.

## Klasör Yapısı (Frontend)

```
frontend/src/
  pages/            Rotalara karşılık gelen üst düzey ekranlar
  components/       Yeniden kullanılabilir UI parçaları (sayfaya özel alt klasörlerde)
  api/              Backend ile konuşan fetch sarmalayıcıları (her kaynak için bir dosya)
  state/            Zustand store'ları (auth, UI durumu)
  types/            Backend şemalarıyla birebir eşleşen TS tipleri
  hooks/            React Query tabanlı veri çekme hook'ları
```

## Kararlar ve Gerekçeleri

- **SQLite**: Yerel geliştirme için sıfır kurulum; Alembic ile şema Postgres'e
  taşınabilir şekilde tutulur.
- **React Query**: Sunucu durumunu (server state) yönetmek için — manuel
  `useEffect` + `fetch` tekrarını önler, cache/refetch mantığını hazır sağlar.
- **Zustand**: Sadece istemci tarafı UI durumu (auth token, tema) için; global
  state karmaşasını önlemek adına minimal tutulur.
- **Structured Outputs**: Claude'dan JSON şema ile yapılandırılmış çıktı istenir;
  backend bunu Pydantic ile tekrar doğrular (AI çıktısına asla körü körüne güvenilmez).
