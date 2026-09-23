# ELİF OS — Sürüm Notları

## v0.1.0 — İlk Çalışan Sürüm (2026-08-28)

İlk uçtan uca çalışan sürüm. Aşağıdaki akış baştan sona gerçek verilerle
test edildi:

Kayıt ol → Giriş yap → Görev oluştur → WHAT NOW ile öneri al → Odak Modu'nu
başlat → Tamamla → Görev listesinde ve Haftalık Değerlendirme'de yansısın.

### Eklenenler

- Kimlik doğrulama (kayıt, giriş, JWT access/refresh, korumalı rotalar)
- Görev yönetimi: oluşturma, düzenleme, tamamlama, yeniden açma, erteleme,
  silme, filtreleme, arama, sıralama
- Brain Dump: AI ile serbest metinden taslak görev çıkarma + onay akışı
  (yapılandırılmadıysa net "AI yapılandırılmadı" mesajı)
- Hedef → Proje → Kilometre Taşı hiyerarşisi + otomatik ilerleme hesaplama
- Akademik modül: dersler, konular, ödev/sınavlar, çalışma oturumları,
  "Ne Çalışmalıyım?" önerisi
- Deadline Intelligence: risk seviyesi hesaplama (kural tabanlı, LLM'siz)
- Günlük Planlayıcı: kural tabanlı zamanlama motoru, sürükle-bırak yeniden
  zamanlama, manuel blok ekleme
- WHAT NOW: skorlama algoritması + isteğe bağlı AI açıklaması
- Odak Modu: gerçek zamanlı sayaç, gerçek süre kaydı
- Enerji takibi
- Haftalık Değerlendirme: gerçek istatistikler + (AI varsa AI, yoksa kural
  tabanlı) gözlemler
- AI Asistan: tool-calling ile görev/plan/hedef verisine erişim, yıkıcı
  işlemler için onay akışı
- AI Belleği: kullanıcı tarafından görülebilir/silinebilir tercih kayıtları
- Google Calendar OAuth entegrasyonu (kod tam; kimlik bilgisi bekleniyor)
- Bildirimler: riskli son tarih, kaçırılan oturum, hedef durgunluğu tespiti
- Ayarlar: profil, çalışma saatleri, tema, takvim bağlantısı, AI belleği
- Açık/koyu/sistem teması, mobil uyumlu arayüz
- Backend: 27 pytest testi (auth, task CRUD, zamanlama, deadline, hedef
  hiyerarşisi, bildirimler, şifreleme, hız sınırlama)
- Frontend: 8 Vitest testi (UI bileşenleri, Brain Dump akışı, WHAT NOW)
- Güvenlik: bcrypt, JWT, sahiplik kontrolü, Google token şifreleme, auth
  hız sınırlaması, global hata yönetimi

### Bilinen Sınırlamalar

- Google Calendar ve Anthropic API canlı olarak test edilemedi (kimlik
  bilgisi gerektirir) — sadece "yapılandırılmadığında" davranış doğrulandı.
- AI uç noktalarına (chat, brain-dump) rate limiting henüz eklenmedi.
- Recharts kütüphanesi kuruldu ancak ayrı bir "Analitik" sayfası henüz
  yazılmadı — mevcut ilerleme çubukları (Recharts olmadan) Dashboard,
  Goals ve Weekly Review sayfalarında gösteriliyor.
- E-posta doğrulama / şifre sıfırlama akışı yok.

Bkz. [ROADMAP.md](ROADMAP.md) için detaylı, kalem kalem durum takibi.
