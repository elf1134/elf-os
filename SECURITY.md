# ELİF OS — Güvenlik

Bu belge, geliştirme sırasında yapılan güvenlik denetiminin sonuçlarını ve
bilinen sınırlamaları şeffaf şekilde listeler.

## Kimlik Doğrulama ve Yetkilendirme

- Şifreler `bcrypt` ile hash'lenir (72 byte'a kesilir — bcrypt'in doğal
  sınırı; `app/core/security.py`). Düz metin şifre asla saklanmaz veya
  loglanmaz.
- JWT access token (60 dk) ve refresh token (30 gün) ayrımı yapılır; her
  token'ın `type` alanı doğrulanır (bir refresh token access token yerine
  kullanılamaz).
- Her kaynak uç noktası (`/api/tasks`, `/api/goals`, ...) sorguyu
  `user_id == current_user.id` ile filtreler; başka bir kullanıcının kaydına
  erişim 404 döner (varlığı bile sızdırmaz). Bkz.
  `tests/test_tasks.py::test_cannot_access_other_users_task`.
- Giriş/kayıt uç noktalarına bellek içi hız sınırlaması uygulanır (60
  saniyede IP başına 10 istek) — brute-force'u yavaşlatır. **Sınırlama:** tek
  süreçli bellek içi bir çözümdür; çoklu worker/üretim dağıtımında Redis
  tabanlı bir çözüme (ör. `slowapi` + Redis) geçilmelidir.

## Girdi Doğrulama

- Tüm API girdileri Pydantic şemalarıyla doğrulanır (tip, uzunluk, format).
- SQL enjeksiyonu: Ham SQL string birleştirme hiçbir yerde kullanılmaz; tüm
  sorgular SQLAlchemy ORM üzerinden parametrize edilir.
- XSS: Frontend React kullanır (varsayılan olarak kaçışlanır);
  `dangerouslySetInnerHTML` kod tabanının hiçbir yerinde kullanılmaz.
- CSRF: Kimlik doğrulama çerezler yerine `Authorization: Bearer` header'ı ile
  yapılır — klasik CSRF saldırı yüzeyi bu nedenle uygulanabilir değildir.

## Sırlar ve Ortam Değişkenleri

- `backend/.env` `.gitignore` içindedir, hiçbir gerçek anahtar repoya
  commit'lenmemiştir. `.env.example` sadece placeholder değerler içerir.
- `SECRET_KEY` varsayılan (güvensiz) değerde bırakılırsa, uygulama
  başlangıcında bir uyarı loglanır (`app/main.py`).
- API anahtarları (`ANTHROPIC_API_KEY`, `GOOGLE_CLIENT_SECRET`) yalnızca
  backend ortam değişkenlerinde tutulur; frontend'e HİÇBİR ZAMAN gönderilmez.

## Google OAuth Token'ları

- Access/refresh token'lar veritabanında **şifrelenmiş** saklanır
  (`app/core/crypto.py`, Fernet simetrik şifreleme, anahtar `SECRET_KEY`'den
  türetilir). Veritabanı dosyasına erişen biri token'ları düz metin olarak
  göremez.
- **Sınırlama:** Şifreleme anahtarı `SECRET_KEY`'den türetildiği için,
  `SECRET_KEY` değiştirilirse önceden şifrelenmiş token'lar çözülemez hale
  gelir ve kullanıcı takvimi yeniden bağlamalıdır. Üretimde `SECRET_KEY`'i
  sabitleyin ya da ayrı bir `ENCRYPTION_KEY` tanımlayın.

## AI Güvenliği

- **AI veritabanını asla doğrudan değiştirmez.** Brain Dump, Claude'dan
  zorunlu bir `tool_choice` ile yapılandırılmış JSON ister; bu çıktı
  Pydantic ile tekrar doğrulanır ve KULLANICI ONAYLAMADAN kaydedilmez
  (`app/ai/brain_dump.py`, `app/api/ai.py::brain_dump`).
- AI Asistan'ın tüm eylemleri (`app/ai/tools.py::execute_tool`) normal
  API router'larının kullandığı aynı doğrulanmış model/servis katmanından
  geçer — asistan ham SQL çalıştıramaz.
- Yıkıcı işlemler (`delete_task`) `confirm=true` olmadan ÇALIŞTIRILMAZ; araç
  önce bir onay isteği döner, gerçek silme yalnızca kullanıcı açıkça onay
  verdikten sonraki çağrıda gerçekleşir.
- **Prompt injection değerlendirmesi:** Asistanın tool sonuçlarında (ör. bir
  görev başlığında) kötü niyetli talimat içeren metin geri gelebilir. Ancak
  tüm araçlar yalnızca o anki kimliği doğrulanmış kullanıcının KENDİ verisi
  üzerinde, sahiplik kontrolüyle (`task.user_id == user.id`) çalışır — bu
  nedenle böyle bir enjeksiyonun olası etkisi kullanıcının kendi verisiyle
  sınırlıdır (ör. kendi görevini yanlışlıkla tamamlanmış işaretlemek).
  Çok kullanıcılı/paylaşılan veri senaryoları için ek katman gerekir.
- AI çıktısı asla ham HTML olarak render edilmez (yalnızca düz metin olarak
  gösterilir), bu da AI çıktısı üzerinden XSS'i engeller.

## Hata Yönetimi

- Global exception handler (`app/main.py`) beklenmeyen hataları kullanıcıya
  ham stack trace olarak GÖSTERMEZ; genel bir mesaj döner, gerçek hata sunucu
  loglarına yazılır.
- Frontend, API hatalarını `extractErrorMessage` ile güvenli, kullanıcı
  dostu mesajlara çevirir (`frontend/src/lib/api.ts`).

## Rate Limiting

Şu anda yalnızca `/api/auth/login` ve `/api/auth/register` sınırlanır. Diğer
uç noktalar (ör. `/api/ai/chat`) sınırlanmamıştır — bu, yerel/tek kullanıcılı
geliştirme için kabul edilebilir, ancak birden fazla kullanıcıya açık bir
dağıtımda tüm uç noktalara (özellikle AI çağrıları maliyetli olduğu için)
sınırlama eklenmesi ÖNERİLİR.

## Bilinen Sınırlamalar (özet)

| Alan | Durum | Öneri |
|---|---|---|
| Rate limiting | Yalnızca auth uç noktaları | Üretimde tüm uç noktalara Redis tabanlı limitleme |
| Google token şifreleme anahtarı | `SECRET_KEY`'den türetilir | Üretimde ayrı, rotasyonu yönetilen bir anahtar |
| AI uç noktaları rate limit | Yok | Kötüye kullanımı/maliyeti sınırlamak için eklenmeli |
| CORS | Tek origin (`FRONTEND_ORIGIN`) | Çoklu ortam için origin listesi genişletilmeli |
| Audit log | Yok | Kim neyi ne zaman değiştirdi kaydı üretim için eklenebilir |

## Denetim Sırasında Bulunan ve Düzeltilen Sorunlar

1. Google OAuth token'ları başlangıçta düz metin saklanıyordu →
   Fernet şifreleme eklendi.
2. Giriş/kayıt uç noktalarında hız sınırlaması yoktu → eklendi.
3. `SECRET_KEY` varsayılan değerde kalırsa sessizce devam ediyordu →
   başlangıç uyarısı eklendi.
