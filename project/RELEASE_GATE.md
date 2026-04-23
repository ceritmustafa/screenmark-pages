# ScreenMark Release Gate

> Son güncelleme: 2026-04-21

## Hard Blockers — Kod Düzeyinde

| # | Kontrol | Durum |
|---|---------|-------|
| 1 | `swift build --product ScreenMarkApp` hatasız geçiyor | ✅ |
| 2 | `swift test` 206/206 geçiyor | ✅ |
| 3 | Lifetime/subscription/trial flow testlerde kapsanıyor | ✅ |
| 4 | Overlay production build'de dev-only UI içermiyor | ✅ (Sprint 1) |
| 5 | `Info.plist` LSUIElement = true (Dock'ta görünmez) | ✅ (Sprint 6) |
| 6 | Versiyon 1.0.0 | ✅ (Sprint 6) |
| 7 | `ScreenMark.entitlements` oluşturuldu | ✅ (Sprint 6) |

## Hard Blockers — Submission İçin Açık

| # | Kontrol | Durum |
|---|---------|-------|
| 8 | Bundle ID `com.mustafacerit.screenmark` → App Store Connect'teki gerçek ID ile eşleşiyor | ⚠️ Doğrulanmadı |
| 9 | StoreKit ürün ID'leri (`screenmark_pro_*`) App Store Connect'te tanımlı | ⚠️ Doğrulanmadı |
| 10 | Archive + codesign + notarize pipeline test edildi | ⚠️ Dry-run doğrulandı, gerçek cert bekliyor |
| 11 | `insecure-defaults` security audit tamamlandı | ✅ Temiz (2026-04-14; 2026-04-20 tazelendi — kullanılmayan `FileTimestamp` PrivacyInfo reason'u çıkarıldı) |
| 12 | Shipped .app localization doğrulandı (8 dil gerçekten yüklü) | ✅ 2026-04-20 (SUB-009 fix) |

## MAS Submission — Cert Geldiğinde Yapılacaklar

> `Scripts/archive_mas.sh` preflight + DRY_RUN modu ile hazır. Son dry-run: 2026-04-20 (8 lproj + PrivacyInfo + entitlements embedded; ad-hoc sign verify OK). Gerçek submission için aşağıdaki sıra takip edilecek.

1. **Apple Developer Program** üyeliği aktif (yıllık $99)
2. **App Store Connect'te app record oluştur**:
   - Bundle ID: `com.mustafacerit.screenmark`
   - SKU: `screenmark-macos-1`
   - Primary language: Turkish veya English
3. **StoreKit ürünleri tanımla** (In-App Purchases sekmesi):
   - `screenmark_pro_lifetime` — Non-Consumable
   - `screenmark_pro_monthly` — Auto-Renewable Subscription
   - `screenmark_pro_yearly` — Auto-Renewable Subscription
   - Tümü "Ready to Submit" durumuna getir
4. **Sertifikaları Keychain'e yükle** (ASC → Certificates → Mac App Store):
   - `3rd Party Mac Developer Application: Mustafa Cerit (TEAMID)`
   - `3rd Party Mac Developer Installer: Mustafa Cerit (TEAMID)`
5. **Provisioning profile indir** (ASC → Profiles → Mac App Store Distribution):
   - `screenmark_mas.provisionprofile` → `~/Library/MobileDevice/Provisioning Profiles/`
6. **Script'i çalıştır** (env var'larla gerçek TEAMID + profile path geçir):
   ```bash
   APP_IDENTITY="3rd Party Mac Developer Application: Mustafa Cerit (TEAMID)" \
   PKG_IDENTITY="3rd Party Mac Developer Installer: Mustafa Cerit (TEAMID)" \
   PROVISIONING_PROFILE="$HOME/Library/MobileDevice/Provisioning Profiles/screenmark_mas.provisionprofile" \
   ./Scripts/archive_mas.sh
   ```
7. **`.pkg`'yi validate et**:
   ```bash
   xcrun altool --validate-app -f dist/ScreenMark.pkg --type osx -u <apple-id>
   ```
8. **Upload** (Transporter.app veya `altool --upload-app`)
9. **Review'a gönder** (ASC → App → Add for Review)

## Sandbox Uyumluluk (2026-04-14 analizi)

| Bileşen | Durum |
|---------|-------|
| Screen Recording (`SCStream`) | ✅ TCC permission ile sandbox uyumlu |
| Global Hotkeys (`NSEvent.addGlobalMonitorForEvents`) | ✅ Accessibility TCC ile sandbox uyumlu |
| Carbon Hotkeys (`RegisterEventHotKey`) | ✅ Sandbox uyumlu |
| Launch at Login (`SMAppService`) | ✅ Sandbox uyumlu |
| Mikrofon (`AVFoundation`) | ✅ Entitlement eklendi: `com.apple.security.device.audio-input` |

## Manuel Test Matrisi (2026-04-18, Sprint 6c eklemeleri 2026-04-21)

| Senaryo | Durum |
|---------|-------|
| Tek ekranda overlay davranışı | ✅ |
| Çoklu ekranda overlay davranışı | ✅ |
| Freeze, zoom, whiteboard, snip, snapshot export | ✅ (freeze wrong-screen bug + ESC focus fix eklendi) |
| Recording başlatma/durdurma + çıktı doğrulama | ✅ |
| Free kullanıcı kilitli özellik mesajlaşması | ✅ |
| Purchase restore yeniden başlatmada | ✅ (ASC bağlantısı yok — no-op olarak doğrulandı, crash yok, yerel state korunuyor) |
| Accessibility + klavye-only navigasyon | ✅ (Tab/Space/ESC çalışıyor; tam VoiceOver denetimi submission sonrası iyileştirilecek) |
| Onboarding ilk açılışta gösteriyor | ✅ (copy düzeltildi: "Freeze & Capture" → "Freeze & Snip") |
| Trial başlatma + 5/7 gün bildirimleri | ✅ (anlık "Trial started" geliyor; 5/7 gün scheduler unit test'te kapsanıyor) |
| Shortcut recorder atanmış hotkey'i yakalıyor (BUG-001) | ⚠️ Release build'de doğrulanacak (unit test yeşil) |
| Classic zoom Pro hesabında freeze-backed (BUG-002) | ⚠️ Release build'de doğrulanacak (unit test yeşil) |
| 60fps recording smooth playback (BUG-003) | ⚠️ Release build'de doğrulanacak |
| 1440p/30fps 60s kayıt ≤ ~90 MB (BUG-004) | ⚠️ Release build'de doğrulanacak |
| Built-in mic 30s konuşma duyulabilir (BUG-005) | ⚠️ Release build'de doğrulanacak (soft-clip unit test yeşil) |
| ESC drawing + mode'u tek basışta temizliyor (BUG-006) | ✅ Unit test (AppCoordinatorTests) |
| Dual-monitor köprü snip doğru composit (BUG-007) | ⚠️ Release build'de doğrulanacak (SnapshotExporter unit test yeşil) |
| Freeze A + mouse B → snapshot B (BUG-008) | ⚠️ Release build'de doğrulanacak (AppCoordinator+Snapshot unit test yeşil) |
| Recording/snip bildirimine tıklayınca Finder açılıyor (BUG-009) | ⚠️ Release build'de doğrulanacak (NotificationService + Commerce handler unit test yeşil) |

## Submission Notları

- **Minimum macOS**: 15.0 (`Info.plist: LSMinimumSystemVersion = 15.0`) ✅
- **Bundle ID**: Signing sertifikası ve App Store Connect hesabıyla hizalanmalı
- **StoreKit product ID'leri**: Sandbox ortamında test edilmeli
- **Archive yolu**: `Scripts/archive_mas.sh` ile belgeli; gerçek signing/provisioning ile doğrulanmalı
- **Privacy Policy URL**: `https://ceritmustafa.github.io/ScreenMark/privacy/` hazır

## Security Audit Özeti (2026-04-14)

| Kontrol | Sonuç |
|---------|-------|
| Hardcoded secrets/credentials | ✅ Yok |
| UserDefaults'ta hassas veri (transaction/receipt) | ✅ Yok — entitlements JSON dosyada, sadece AppSettings + boolean'lar UserDefaults'ta |
| Staging/localhost URL'leri | ✅ Yok |
| Force unwrap production kodunda | ✅ Yok |
| Raw print() ile sensitive data | ✅ Yok |
| StoreKit revocationDate kontrolü | ✅ Var (`CommerceService.swift:269`) |
| StoreKit unverified transaction koruması | ✅ Var — `verified()` fonksiyonu `.unverified` durumunda throw eder |
| Temp dosya yönetimi | ✅ ApplicationSupportDirectory kullanılıyor |
