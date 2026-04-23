# ScreenMark — Lessons Learned

> Çözülen sorunlar, bilinen tuzaklar ve mimari kararların gerekçeleri.
> Son güncelleme: 2026-03-23 (Phase 2)

---

## 1. Overlay Window Sync Race Condition (2026-03-23)

### Problem
Menu bar popover'dan mod açıldığında (Zoom, Draw, Snip, Freeze, vb.) ESC tuşu modu kapatmıyordu. Klavye kısayoluyla açılınca sorun yoktu.

### Kök Neden
`AppCoordinator.handle(_:)` içindeki özel handler'lar (`toggleZoom`, `toggleFreeze`, `toggleBoard`, vb.) `overlayWindowManager.sync()` çağırmıyordu. Sync yalnızca `default:` case'inde ve async Combine subscriber üzerinden çalışıyordu.

**Yarış koşulu zaman çizelgesi:**
```
T=0ms: toggleFreeze() çağrılır (sceneState güncellenir)
T=1ms: focusWindow() çağrılır (pencere henüz görünmez, alphaValue=0)
T=5ms: RunLoop iterasyonu biter
T=6ms: Combine subscriber tetiklenir → sync() çalışır → pencere görünür olur
       AMA: firstResponder zaten ayarlanmış, key event yönlendirmesi bozuk
```

### Çözüm
- `overlayWindowManager.sync()` çağrısını `handle()` içinde switch'ten SONRA, `focusWindow()`'dan ÖNCE taşıdık — tüm action'lar için çalışır.
- `exitActiveModes()` içinde her branch'e `sync()` eklendi.
- `startCountdown()` içinde `focusWindow()` öncesine `sync()` eklendi.
- Menu bar popover `handle()` başında `closePopover()` ile kapatılıyor.

### Ders
**Asla async subscriber'a güvenme** — kritik window state değişikliklerini senkron yap. Combine `receive(on: RunLoop.main)` bir sonraki RunLoop iterasyonuna erteliyor, bu da focusWindow ile sync arasında yarış yaratıyor.

---

## 2. Multi-Monitor Overlay Window Yönetimi

### Mimari
Her `NSScreen` için ayrı `OverlayWindow` oluşturuluyor. `preferredDisplayID()` cursor konumuna göre doğru ekranı seçiyor. Koordinat dönüşümü `globalPoint(from:for:)` / `localPoint(_:for:)` ile yapılıyor.

### Tuzak
Overlay penceresi `alphaValue=0` ile başlıyor ve yalnızca `sync()` çağrısıyla görünür hale geliyor. Sync çağrılmadan `focusWindow()` yapılırsa, görünmez pencereye focus verilir → kullanıcı etkileşimi çalışmaz.

### Ders
Window lifecycle sırası: **state güncelle → sync (visibility/mouse) → focus (key window)**. Bu sıra asla bozulmamalı.

---

## 3. NSPopover ve Focus Çakışması

### Problem
Menu bar popover `.transient` davranışıyla otomatik kapanmalıydı, ama `coordinator.handle()` çağrısı popover kapatılmadan önce çalışıyordu. Popover açıkken overlay window focus alamıyordu.

### Çözüm
`handle()` fonksiyonunun en başında `statusItemController.closePopover()` çağırarak popover'ı mode activation'dan önce kapatıyoruz.

### Ders
**UI bileşenleri arası focus çakışması**: Bir pencereye focus vermeden önce, focus'u çalabilecek tüm diğer UI bileşenlerini (popover, sheet, panel) kapat.

---

## 4. ESC Tuşu Çoklu İşleme Katmanları

### Mimari
ESC üç aşamada yakalanıyor:
1. **OverlayHostingView.keyDown()** — pencere focus'taysa
2. **Local key monitor** (`NSEvent.addLocalMonitorForEvents`) — app aktifse
3. **Global key monitor** (`NSEvent.addGlobalMonitorForEvents`) — fallback

### Tuzak
Global monitor `Task { @MainActor in ... }` kullanıyor — bu bir RunLoop gecikmesi ekliyor. Overlay window düzgün focus almadıysa, yalnızca global monitor'a düşülüyor ve gecikme yaşanıyor.

### Ders
Birincil ESC yolu (OverlayHostingView) her zaman çalışmalı. Global monitor yalnızca güvenlik ağı olarak kalmalı.

---

## 5. ScreenCaptureKit vs CGWindowListCreateImage

### Karar
`SCScreenshotManager.captureImage()` kullanılıyor, eski `CGWindowListCreateImage` değil.

### Gerekçe
- Explicit `displayID` seçimi ile multi-monitor desteği daha sağlam
- ScreenMark app'ini capture'dan hariç tutma (`excludingApplications`)
- Scale factor desteği (Retina)
- macOS 14+ gerekliliği zaten var

---

## 6. Combine Subscriber Kullanım Kuralları

### Kural
`machine.$sceneState.receive(on: RunLoop.main).sink { ... }` **ikincil güvenlik ağı** olarak çalışmalı, **birincil sync mekanizması değil**.

### Neden
`receive(on: RunLoop.main)` mevcut RunLoop iterasyonunda değil, bir sonrakinde çalışır. Bu, senkron kod ile arasında gözle görülmez ama davranışı bozan bir gecikme yaratır.

### Uygulama
Her `machine.handle()` / `machine.setXxx()` çağrısından sonra `overlayWindowManager.sync()` açıkça çağır. Subscriber yedek olarak kalır.

---

## 7. Draw Modda Text Element Tekrar Seçimi (2026-03-23)

### Problem
Draw modda text eklendikten ve seçim bırakıldıktan sonra, text'ler tekrar seçilemiyordu.

### Kök Neden
`canManipulateTextElements` sadece `selectedTextElementID != nil` kontrolü yapıyordu. Seçim bırakıldığında ID nil olunca, drag/tap handler'lar text hit-test'e hiç düşmüyordu.

### Çözüm
`hasSelectableTextElements` computed property eklendi: draw modda scene'de text varsa true. Bu sayede seçim olmadan da text'lere tıklayıp tekrar seçilebilir hale geldi. Drag handler'da text hit-test yapılıp, miss durumunda normal çizime fall-through sağlandı.

### Ders
**Guard koşulları zincirleme davranışı kırar**: Bir bileşenin etkileşim modunu kontrol eden guard, tüm alt senaryoları kapsamalı. `canManipulate` "seçili olan ile etkileşim" için doğruydu ama "seçim başlatma" senaryosunu dışarıda bırakıyordu.

---

## 8. NSPopover Transient Davranışı (2026-03-23)

### Problem
`.transient` popover davranışı bazı durumlarda otomatik kapanmıyordu.

### Çözüm
Ek olarak global mouse click monitor ve 3 saniyelik inactivity timer eklendi. Bu ikili mekanizma popover'ın her durumda kapanmasını garanti eder.

### Ders
macOS'un `.transient` popover davranışı her senaryoda güvenilir değil. Kendi dismiss mekanizmanızı ekleyin.

---

## 9. @State Scope ve Nested View'lar (2026-03-23)

### Problem
`OverlaySceneView`'daki `@State` değişkenler `OverlayElementsLayer` (farklı struct) içinden erişilemiyordu.

### Çözüm
Resize handle'ı kendi `@State`'ine sahip ayrı bir `TextResizeHandle` view'ına çıkardık.

### Ders
SwiftUI'da `@State` yalnızca tanımlandığı view ve child closure'ları içinde geçerlidir. Farklı struct'lar arası paylaşım gerekiyorsa ya ayrı view oluştur ya da `@Binding` kullan.
