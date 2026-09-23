# ELİF OS — Öğrenme Rehberi

Bu belge, yazılım geliştirmeyi öğrenirken bu kod tabanını okuyan biri için
yazıldı. Amaç, "neden böyle yapıldı"yı anlamanıza yardımcı olmak.

## 1. Büyük Resim

İki bağımsız uygulama, HTTP üzerinden konuşuyor:

```
Tarayıcı  ⇄  Frontend (React, :5173)  ⇄  HTTP/JSON  ⇄  Backend (FastAPI, :8000)  ⇄  SQLite
```

Bu ayrım önemlidir: frontend hiçbir zaman veritabanına doğrudan erişmez,
her zaman backend'in doğruladığı bir API üzerinden geçer. Bu, "asla AI'ın
veritabanını doğrudan değiştirmemesi" kuralını da mümkün kılan aynı prensip.

## 2. Backend'i Okuma Sırası

Backend'i ilk kez okuyorsanız şu sırayla ilerleyin:

1. `app/core/config.py` — hangi ayarlar var, nereden geliyor?
2. `app/core/database.py` — veritabanı bağlantısı nasıl kuruluyor?
3. `app/models/user.py` — en basit model, SQLAlchemy söz dizimini gösterir.
4. `app/models/task.py` — ilişkiler (`relationship`, `ForeignKey`) burada.
5. `app/schemas/task.py` — Pydantic şemaları modellerden NEDEN ayrı?
   (Model = veritabanı satırı. Şema = API'nin dışarıya gösterdiği/kabul
   ettiği şekil. Bir modeldeki her alanı dışarı vermek istemeyebilirsiniz —
   ör. `hashed_password` asla bir şemada yer almaz.)
6. `app/api/tasks.py` — bir router'ın gerçek akışı: istek gelir → Pydantic
   doğrular → veritabanı sorgusu → yanıt şeması ile dönülür.
7. `app/services/scheduling.py` — "iş mantığı" router'dan neden ayrı bir
   dosyada? Çünkü hem API hem de AI asistan aynı mantığı çağırabilmeli, kod
   tekrarlanmamalı.

## 3. En Öğretici Üç Dosya

**`app/services/what_now.py`** — WHAT NOW algoritması. Burada "yapay zeka"
aslında sıradan Python if/else ve toplama işlemleridir (bkz. `PRIORITY_WEIGHTS`,
skor toplama). LLM sadece SONUCU daha güzel cümlelerle anlatmak için
kullanılır (`app/ai/explain.py`). Bu ayrım kasıtlı: kritik kararları
belirleyici (deterministic), test edilebilir kodda tutuyoruz.

**`app/ai/tools.py`** — bir AI asistanının "araçları" nasıl güvenli
tutulur? Her fonksiyon (`create_task`, `delete_task`, ...) normal API'nin
kullandığı aynı SQLAlchemy modellerini, aynı sahiplik kontrolünü kullanır.
AI'a "güç" değil, önceden tanımlanmış, sınırlı bir menü veriyoruz.

**`frontend/src/components/tasks/BrainDumpModal.tsx`** — bir "onay akışı"nın
frontend'de nasıl modellendiği: `drafts` state'i `null` iken metin girme
ekranı, `drafts` doluyken düzenlenebilir önizleme ekranı gösterilir. Hiçbir
API çağrısı "taslak → gerçek kayıt" arasında otomatik geçiş yapmaz — kullanıcı
her zaman "Onayla ve Kaydet" butonuna basmalıdır.

## 4. Frontend'i Okuma Sırası

1. `src/types/index.ts` — backend şemalarıyla birebir eşleşen TS tipleri.
   Backend bir alan eklerse/değiştirirse, burada da güncellenmeli.
2. `src/lib/api.ts` — her isteğe otomatik token ekleyen ve 401'de token
   yenileyen axios "interceptor" deseni.
3. `src/api/tasks.ts` gibi dosyalar — backend uç noktalarının ince
   sarmalayıcıları. Bileşenler asla doğrudan `fetch`/`axios` çağırmaz.
4. `src/pages/Tasks.tsx` — TanStack Query'nin `useQuery`/`useMutation`
   deseni: `useQuery` veri ÇEKMEK için, `useMutation` veri DEĞİŞTİRMEK için.
   `onSuccess` içinde `invalidateQueries` çağrısı, "bu veri değişti, önbelleği
   tazele" demektir.
5. `src/state/authStore.ts` — Zustand ile minimal global state. Neden Redux
   değil? Bu proje ölçeğinde tek bir "kim giriş yapmış" ve "tema ne"
   bilgisini tutmak için Redux'un karmaşıklığına gerek yok.

## 5. Neden Bu Kütüphaneler?

| Seçim | Alternatif | Neden bu seçildi |
|---|---|---|
| FastAPI | Flask, Django | Otomatik Pydantic doğrulama + otomatik OpenAPI dokümantasyonu (`/docs`) |
| SQLAlchemy + Alembic | Ham SQL | Tip güvenliği + şema değişikliklerinin geçmişi (migration) |
| TanStack Query | Sadece `useEffect` + `fetch` | Cache, otomatik yeniden deneme, "stale" veri yönetimi hazır gelir |
| Zustand | Redux | Çok daha az "boilerplate" kod, bu proje ölçeğinde yeterli |
| Tailwind CSS | Sıfırdan CSS | Tasarım tutarlılığı (aynı boşluk/renk skalası) hızlı sağlanır |

## 6. Bir Özelliği Uçtan Uca Takip Etmek İçin Egzersiz

"Bir görevi tamamlamak" özelliğini takip edin:

1. Kullanıcı arayüzde ✓ butonuna tıklar → `frontend/src/pages/Tasks.tsx`
   içindeki `completeMutation`.
2. `frontend/src/api/tasks.ts::tasksApi.complete` → `POST /api/tasks/{id}/complete`.
3. `backend/app/api/tasks.py::complete_task` → `task.status` ve
   `task.completed_at` güncellenir.
4. `onSuccess` içinde `qc.invalidateQueries(["tasks"])` → liste otomatik
   yeniden çekilir, ekran güncellenir.

Bu döngüyü anladıysanız, uygulamadaki hemen hemen her CRUD özelliği aynı
kalıbı izler.

## 7. Sıradaki Adımlar (Kendi Kendinize Denemeler)

- `Category` modeline bir `icon` alanı ekleyip uçtan uca (model → şema →
  router → frontend tipi → form) geçirmeyi deneyin.
- `services/what_now.py` içindeki skorlama ağırlıklarını değiştirip
  `tests/test_scheduling.py` testlerinin hâlâ geçtiğinden emin olun.
- Yeni bir Pydantic şeması yazıp yanlış bir tip gönderdiğinizde FastAPI'nin
  otomatik olarak nasıl 422 hatası döndürdüğünü `http://localhost:8000/docs`
  üzerinden gözlemleyin.
