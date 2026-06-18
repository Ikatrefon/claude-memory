---
name: reference-mobile-web-gotchas
description: "Pułapki mobilnych gier webowych: orientation lock wymaga fullscreena (Android), in-app WebView wyłącza fullscreen/DeviceOrientation/screen.orientation"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 42ecb71f-295d-47b5-b01f-77be0ea7d00f
---

# Mobilne gry web — pułapki orientacji i czujników

## Blokada orientacji (portret)
- `screen.orientation.lock('portrait-primary')` na **Androidzie wymaga trybu fullscreen** (`element.requestFullscreen()`). Bez fullscreena bywa po cichu ignorowane (lock „nie trzyma").
- Wzorzec: `requestFullscreen()` z gestu → lock w `fullscreenchange`. Wołać `requestFullscreen` PRZED lockiem może „zjeść" user-activation jeśli przeglądarka odrzuci fullscreen → lock leci bez gestu i jest ignorowany.
- Arkanoid 3D „blokuje" pion bo: lock (best-effort) + nakładka `#landscape-warn` „Rotate to portrait" + user tiltuje delikatnie (nie przekracza progu autoobrotu). Pod mocnym przechyłem lock też by nie utrzymał.

## In-app WebView = zabójca (KLUCZOWE)
Linki z QR/skanera/aparatu/Messengera często otwierają się w **okrojonym in-app WebView**, NIE w pełnym Chrome/Samsung Internet. Tam typowo:
- `requestFullscreen()` → odrzucone (FS:NIE),
- `screen.orientation.angle` → utknięte na 0 (brak raportu obrotu),
- **`DeviceOrientation`/`devicemotion` mogą w ogóle nie dochodzić** (tilt nie działa),
- ekran i tak obraca się z systemem, a strona nie ma jak tego wykryć/naprawić.
Objaw na Samsung S24 (PICK&MIX): FS:NIE, kąt:0, brak tiltu — mimo „aparatu Samsunga". To środowisko, nie kod.

## Konsekwencje praktyczne
- Tilt + lock działają niezawodnie tylko w **PEŁNYCH przeglądarkach** po HTTPS.
- Dla gry z QR rozważyć: nudge „otwórz w przeglądarce", detekcja WebView (UA zawiera `; wv)` lub brak fullscreena), kontr-obrót CSS **sterowany wymiarami okna** (nie `screen.orientation.angle`, który w WebView = 0) — działa tylko jeśli WebView aktualizuje innerWidth/innerHeight przy obrocie.
- Diagnostyka na ekranie (status: FS / kąt / wymiary / licznik DeviceOrientation / tag Browser-vs-WebView) szybciej rozstrzyga niż zgadywanie — gdy nie masz fizycznego urządzenia.
- iOS: `DeviceOrientationEvent.requestPermission()` z gestu (Android nie wymaga).
