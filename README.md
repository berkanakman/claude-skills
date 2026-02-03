# Claude Code Skills Library

Bu depo, Claude Code için tasarlanmış kapsamlı bir skill (yetenek) koleksiyonu içermektedir. Skills, Claude'un belirli alanlarda uzmanlaşmış davranışlar sergilemesini sağlayan yapılandırılmış yönergeler sistemidir.

## Genel Bakış

```
skills/
├── _meta/          # Meta-skill'ler (yönetim ve orkestrasyon)
├── web/            # Web geliştirme skill'leri
├── mobile/         # Mobil uygulama skill'leri
├── desktop/        # Masaüstü uygulama skill'leri
└── README.md       # Bu dosya
```

## Skill Kategorileri

### 1. Meta Skills (`_meta/`)

Meta-skill'ler, diğer skill'leri koordine eden ve kalite/güvenlik standartlarını uygulayan üst düzey kontrol mekanizmalarıdır.

| Skill | Açıklama | Kullanıcı Çağrılabilir |
|-------|----------|------------------------|
| `master-governance` | Tüm meta-skill kararlarını koordine eden en üst yönetim katmanı | Hayır |
| `orchestration` | Meta-skill'lerin öncelik sırasını ve etkinleştirme kurallarını yönetir | Hayır |
| `guardrails` | Güvenli, doğru ve bağlama duyarlı davranışı zorunlu kılar | Hayır |
| `qa` | Test yeterliliği ve kalite güvence standartlarını doğrular | Hayır |
| `regression` | Geriye dönük uyumluluk ve değişiklik etkilerini değerlendirir | Hayır |
| `production-readiness` | Üretim ortamına hazırlık kontrollerini yapar | Hayır |
| `canary-feature-flag` | Kademeli dağıtım ve özellik bayrakları stratejisini yönetir | Hayır |
| `cicd-release-gate` | CI/CD pipeline doğrulamalarını gerçekleştirir | Hayır |
| `release-gate` | Son onay ve yayın kapısı kontrollerini uygular | Hayır |
| `migration-only` | Veritabanı migrasyon güvenliği kontrollerini sağlar | Hayır |

#### Meta-Skill Öncelik Sırası

```
1. migration-only      ← Veri kaybı geri alınamaz
2. guardrails          ← Güvenlik pazarlık edilemez
3. regression          ← Mevcut davranış korunmalı
4. production-readiness ← Operasyonel stabilite
5. canary-feature-flag ← Kontrollü dağıtım
6. cicd-release-gate   ← Otomasyon bütünlüğü
7. qa                  ← Kalite güvencesi
8. release-gate        ← Son kontrol listesi
```

---

### 2. Web Development Skills (`web/`)

Web uygulamaları geliştirmek için tasarlanmış skill'ler.

| Skill | Konum | Açıklama | Kullanıcı Çağrılabilir |
|-------|-------|----------|------------------------|
| `api-design` | `web/api/api-design/` | RESTful/RPC API tasarımı, versiyonlama ve güvenlik | Evet |
| `backend-coding` | `web/backend/backend-coding/` | Backend mantığı, ölçeklenebilirlik ve test edilebilirlik | Evet |
| `frontend-coding` | `web/frontend/frontend-coding/` | Frontend kodu, performans ve erişilebilirlik | Evet |
| `web-architecture` | `web/design/web-architecture/` | Web uygulama mimarisi tasarımı | Evet |
| `db-design` | `web/database/db-design/` | Veritabanı şema tasarımı ve optimizasyon | Evet |
| `web-security` | `web/security/web-security/` | Web güvenliği denetimi ve OWASP uyumu | Evet |

#### Backend Coding Yetenekleri
- Katmanlı mimari (Controller → Service → Repository → Model)
- API tasarım kalıpları ve en iyi uygulamalar
- İstek doğrulama ve hata yönetimi
- Kimlik doğrulama ve yetkilendirme
- Yapılandırılmış loglama ve izlenebilirlik
- Önbellek ve asenkron işlem stratejileri

#### Frontend Coding Yetenekleri
- Bileşen mimarisi ve organizasyon
- Durum yönetimi kalıpları
- Performans optimizasyonu (memoization, lazy loading)
- Erişilebilirlik (a11y) standartları
- CSS mimarisi ve metodolojiler
- Test stratejileri

#### API Design Yetenekleri
- REST, GraphQL, gRPC, WebSocket seçimi
- Kaynak adlandırma ve HTTP metod kullanımı
- Versiyonlama stratejileri
- Sayfalama ve filtreleme
- Güvenlik gereksinimleri

#### Web Security Yetenekleri
- OWASP Top 10 kontrol listesi
- Kimlik doğrulama güvenliği
- Girdi doğrulama ve sanitizasyon
- XSS, CSRF, SQL Injection önleme
- Güvenlik başlıkları yapılandırması

---

### 3. Mobile Development Skills (`mobile/`)

Mobil uygulama geliştirmek için tasarlanmış skill'ler.

| Skill | Konum | Açıklama | Kullanıcı Çağrılabilir |
|-------|-------|----------|------------------------|
| `mobile-architecture` | `mobile/architecture/` | Platform bağımsız mobil mimari tasarımı | Evet |
| `mobile-frontend` | `mobile/frontend/` | Mobil UI geliştirme | Evet |
| `mobile-backend` | `mobile/backend/` | Mobil backend entegrasyonu | Evet |
| `mobile-security` | `mobile/security/` | Mobil uygulama güvenliği | Evet |
| `mobile-ui` | `mobile/ui-ux/` | Mobil UI/UX tasarım prensipleri | Evet |

#### Mobile Architecture Yetenekleri
- Mimari kalıplar (MVC, MVP, MVVM, Clean Architecture)
- Katman sorumlulukları (Presentation, Domain, Data)
- Proje yapısı organizasyonu
- Durum yönetimi
- Çevrimdışı-öncelikli tasarım
- Navigasyon kalıpları
- Bağımlılık enjeksiyonu

#### Desteklenen Platformlar
- iOS (Swift, SwiftUI)
- Android (Kotlin, Jetpack Compose)
- Flutter
- React Native

---

### 4. Desktop Application Skills (`desktop/`)

Masaüstü uygulamaları geliştirmek için tasarlanmış skill'ler.

| Skill | Konum | Açıklama | Kullanıcı Çağrılabilir |
|-------|-------|----------|------------------------|
| `desktop-architecture` | `desktop/architecture/` | Electron/Tauri mimari tasarımı | Evet |
| `desktop-ui` | `desktop/ui/` | Masaüstü UI geliştirme | Evet |
| `desktop-backend` | `desktop/backend/` | Masaüstü backend mantığı | Evet |
| `desktop-security` | `desktop/security/` | Masaüstü uygulama güvenliği | Evet |
| `desktop-packaging` | `desktop/packaging/` | Uygulama paketleme ve dağıtım | Evet |

#### Desktop Architecture Yetenekleri
- Process mimarisi (Main/Renderer ayrımı)
- IPC iletişim kalıpları
- Güvenlik yapılandırması
- Pencere yönetimi
- Veri kalıcılığı
- Otomatik güncelleme mekanizması
- Native entegrasyonlar

#### Desteklenen Framework'ler
| Framework | Runtime | Binary Boyutu | Güvenlik |
|-----------|---------|---------------|----------|
| Electron | Chromium + Node | ~150MB | İyi |
| Tauri | System WebView | ~3MB | Daha İyi |
| NW.js | Chromium + Node | ~100MB | Orta |

---

## Skill Dosya Yapısı

Her skill bir SKILL.md dosyası içerir ve aşağıdaki yapıyı takip eder:

```markdown
---
name: skill-name
description: Skill açıklaması
user-invocable: true/false
disable-model-invocation: true/false
---

[Skill içeriği]
```

### YAML Frontmatter Alanları

| Alan | Açıklama |
|------|----------|
| `name` | Skill'in benzersiz adı |
| `description` | Skill'in ne yaptığının kısa açıklaması |
| `user-invocable` | Kullanıcı tarafından doğrudan çağrılabilir mi |
| `disable-model-invocation` | Model otomatik olarak çağırabilir mi |

---

## Risk Değerlendirme Sistemi

Tüm skill'ler zorunlu risk değerlendirmesi içerir:

### Risk Seviyeleri

| Seviye | Açıklama |
|--------|----------|
| LOW | Düşük risk, sınırlı etki |
| MEDIUM | Orta risk, kontrollü etki |
| HIGH | Yüksek risk, geniş etki |

### Etki Kapsamları

| Kapsam | Açıklama |
|--------|----------|
| LOCAL | Yalnızca ilgili bileşen etkilenir |
| PARTIAL | Birden fazla bileşen etkilenir |
| SYSTEMIC | Tüm sistem etkilenir |

### Geri Alma Karmaşıklığı

| Seviye | Açıklama |
|--------|----------|
| EASY | Basit geri alma |
| MODERATE | Orta düzey çaba gerektirir |
| HARD | Karmaşık geri alma süreci |

---

## Karar Akışı

```
┌─────────────────┐
│  Kullanıcı      │
│  İsteği         │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Guardrails     │ ← Her zaman ilk
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Orchestration  │ ← Hangi skill'ler uygulanacak?
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Domain Skills  │ ← Web/Mobile/Desktop
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  QA/Production  │ ← Kalite ve üretim kontrolleri
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Release Gate   │ ← Son onay
└─────────────────┘
```

---

## Kullanım Örnekleri

### Backend Geliştirme
```
Kullanıcı: "Kullanıcı kaydı için bir API endpoint'i oluştur"

Aktif Skill'ler:
1. guardrails → Bağlam doğrulama
2. api-design → Endpoint tasarımı
3. backend-coding → Kod implementasyonu
4. web-security → Güvenlik kontrolü
5. qa → Test gereksinimleri
```

### Mobil Uygulama
```
Kullanıcı: "Çevrimdışı çalışabilen bir mobil uygulama mimarisi tasarla"

Aktif Skill'ler:
1. guardrails → Platform belirleme
2. mobile-architecture → Mimari tasarım
3. mobile-backend → Senkronizasyon stratejisi
4. mobile-security → Veri güvenliği
```

### Desktop Uygulama
```
Kullanıcı: "Electron ile güvenli bir dosya yöneticisi oluştur"

Aktif Skill'ler:
1. guardrails → Gereksinim doğrulama
2. desktop-architecture → Process ayrımı
3. desktop-security → IPC güvenliği
4. desktop-ui → Kullanıcı arayüzü
```

---

## En İyi Uygulamalar

### Genel Kurallar
1. Varsayımda bulunma - Platform, dil veya framework hakkında kullanıcı belirtmeden varsayım yapma
2. Güvenlik öncelikli - Güvenlik her zaman fonksiyonellikten önce gelir
3. Test edilebilirlik - Her kod test edilebilir olmalı
4. Dokümantasyon - Önemli kararlar belgelenmeli

### Kod Kalitesi
- Katmanlı mimari kullan
- Bağımlılık enjeksiyonu uygula
- Hata yönetimini ihmal etme
- Yapılandırılmış loglama kullan
- Anti-pattern'lerden kaçın

### Güvenlik
- Girdi doğrulaması yap
- Çıktı kodlaması uygula
- En az yetki prensibini takip et
- Hassas verileri koruma altına al
- Güvenlik başlıklarını yapılandır

---

## Katkıda Bulunma

Yeni skill eklemek için:

1. Uygun kategoride klasör oluştur
2. SKILL.md dosyası ekle
3. Zorunlu alanları doldur (name, description, user-invocable)
4. Risk değerlendirme bölümü ekle
5. Çıktı formatı tanımla

---

## Lisans

Bu skill koleksiyonu özel kullanım için tasarlanmıştır.

---

## İletişim

Sorular ve geri bildirimler için issue açabilirsiniz.
