# ELİF OS — Yol Haritası

Bu belge neyin **tamamlandığını**, neyin **kısmen** ve neyin **henüz yapılmadığını**
şeffaf şekilde takip eder. "Tamamlandı" işareti, gerçekten çalışır ve test edilmiş
demektir — sahte/placeholder kod bu listeye tamamlandı olarak girmez.

## Faz 0 — İskelet
- [x] Proje klasör yapısı, Git deposu
- [x] ARCHITECTURE.md, ROADMAP.md

## Faz 1 — Backend Temeli
- [x] Ayarlar, veritabanı bağlantısı, güvenlik yardımcıları
- [x] Tüm SQLAlchemy modelleri (User, Task, Category, Goal, Project, Milestone,
      Subject, Topic, Assignment, StudySession, ScheduleBlock, CalendarEvent,
      EnergyEntry, WeeklyReview, Notification, AIConversation, AIMessage,
      AIMemoryEntry)
- [x] Alembic ilk migration (uygulandı, tablolar doğrulandı)
- [x] Kimlik doğrulama (kayıt/giriş/JWT + hız sınırlama)

## Faz 2 — Çekirdek CRUD
- [x] Task CRUD + filtre/sıralama/arama
- [x] Category CRUD
- [x] Goal / Project / Milestone CRUD + ilerleme hesaplama (tarayıcıda uçtan uca test edildi)

## Faz 3 — Akademik Modül
- [x] Subject / Topic / Assignment / StudySession CRUD
- [x] "Ne Çalışmalıyım?" öneri servisi + uç nokta

## Faz 4 — Zeka Katmanı
- [x] Deadline Intelligence servisi (risk hesaplama, testli)
- [x] Scheduling Engine (günlük plan oluşturma, testli, tarayıcıda doğrulandı)
- [x] WHAT NOW skorlama algoritması (testli, tarayıcıda doğrulandı)
- [x] Energy Entry CRUD ve planlamaya etkisi

## Faz 5 — AI Entegrasyonu
- [x] Claude istemcisi + yapılandırılmış çıktı yardımcıları (kod tam;
      ANTHROPIC_API_KEY olmadan çalıştırılamadı, "yapılandırılmadı" davranışı
      doğrulandı)
- [x] Brain Dump ayrıştırma uç noktası (kod tam, aynı nedenle canlı test edilemedi)
- [x] AI Assistant + tool calling (kod tam, aynı nedenle canlı test edilemedi)
- [x] Weekly Review AI gözlemleri + AI yoksa kural tabanlı yedek (yedek davranışı
      tarayıcıda doğrulandı)

## Faz 6 — Entegrasyonlar
- [x] Google Calendar OAuth (kod tam; GCP kimlik bilgisi olmadan "yapılandırılmadı"
      davranışı tarayıcıda doğrulandı; canlı OAuth akışı test edilemedi)
- [x] Notification servisi (proaktif uyarılar, testli)

## Faz 7 — Frontend Çekirdek
- [x] Auth sayfaları (kayıt/giriş), korumalı rota — tarayıcıda uçtan uca test edildi
- [x] Dashboard (bugün ne önemli, WHAT NOW, özet kartlar)
- [x] Task yönetimi ekranı
- [x] Brain Dump akışı (taslak → düzenle → onayla) — UI + birim testleriyle doğrulandı

## Faz 8 — Frontend Genişleme
- [x] Goals → Projects → Milestones ekranları — tarayıcıda uçtan uca test edildi
- [x] Akademik panel + "Ne çalışmalıyım?" özelliği
- [x] Günlük planlayıcı (sürükle-bırak)
- [x] WHAT NOW kartı + Focus Mode (zamanlayıcı) — tarayıcıda uçtan uca test edildi
- [x] Enerji kaydı
- [x] Haftalık Değerlendirme sayfası — tarayıcıda test edildi
- [x] Takvim bağlantı ekranı (Ayarlar içinde)
- [x] AI Asistan sohbet paneli (AI olmadan "yapılandırılmadı" durumu doğrulandı)
- [x] Ayarlar sayfası, açık/koyu tema
- [~] Analitik grafikler — Recharts kuruldu, ilerleme çubukları var; ayrı,
      grafik-ağırlıklı bir Analitik sayfası henüz yazılmadı

## Faz 9 — Kalite
- [x] Backend testleri (pytest, 27 test): auth, task CRUD, hedef hiyerarşisi,
      scheduling, deadline, bildirimler, şifreleme, hız sınırlama
- [x] Frontend testleri (Vitest, 8 test): UI bileşenleri, Brain Dump akışı, WHAT NOW
- [x] Güvenlik denetimi → SECURITY.md (3 gerçek sorun bulundu ve düzeltildi:
      düz metin OAuth token'ları, auth hız sınırlama eksikliği, sessiz
      varsayılan SECRET_KEY)
- [~] Manuel uçtan uca QA — kritik akışlar (auth, task CRUD, goal hiyerarşisi,
      WHAT NOW, Focus Mode, planlayıcı, akademik, weekly review, AI-devre-dışı
      durumları) gerçek tarayıcıda test edildi; Google Calendar canlı akışı ve
      AI canlı akışı kimlik bilgisi olmadığı için test edilemedi

## Bilinen Sınırlamalar (dürüstçe)
- Google Calendar entegrasyonu, kullanıcı kendi GCP OAuth kimlik bilgilerini
  `.env` dosyasına eklemeden çalışmaz ve bu haliyle canlı test edilemedi.
- AI özellikleri `ANTHROPIC_API_KEY` olmadan "AI yapılandırılmadı" hatası döner;
  sahte/otomatik üretilmiş veriyle doldurulmaz. Kod tamdır ama gerçek bir
  API anahtarıyla canlı doğrulama yapılmamıştır.
- Bildirim sistemi ilk sürümde uygulama içi (in-app) çalışır; e-posta/push
  entegrasyonu kapsam dışıdır.
- AI uç noktalarına (chat, brain-dump) henüz rate limiting eklenmedi (yalnızca
  auth uç noktaları sınırlı).
- Ayrı bir "Analitik" grafik sayfası yazılmadı; mevcut ilerleme göstergeleri
  Dashboard/Goals/Weekly Review içine gömülüdür.
- Test kapsamı, kritik akışları kapsayacak şekilde önceliklendirilmiştir; %100
  kapsama hedeflenmemiştir.
