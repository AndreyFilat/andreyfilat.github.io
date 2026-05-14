# WHO Failed — PWA / APK сборка

## Что внутри пакета

- `index.html` — игра (с регистрацией Service Worker)
- `manifest.json` — production-ready PWA manifest
- `sw.js` — Service Worker (требуется PWABuilder для Play Store)
- `icon-192.png`, `icon-512.png` — обычные иконки
- `icon-maskable-192.png`, `icon-maskable-512.png` — adaptive icons для Android (safe-zone в 80%)

## Шаг 1. Опубликуйте на HTTPS-хостинге

PWABuilder работает только с HTTPS URL.

**Vercel** (рекомендую — 1 минута):
```bash
npm i -g vercel
cd /path/to/who-failed-pwa
vercel
```
Получите URL вида `https://who-failed-xxx.vercel.app`.

**Netlify Drop** (без CLI): зайдите на https://app.netlify.com/drop и просто перетащите папку — получите HTTPS-ссылку.

**GitHub Pages**: создайте репозиторий, загрузите файлы, включите Pages в Settings.

## Шаг 2. Проверьте PWA

Откройте опубликованную ссылку → DevTools (F12) → Lighthouse → запустите аудит "Progressive Web App". Должен показать ≥90 баллов.

Если есть ошибки про SW или manifest — сначала исправьте их, потом возвращайтесь в PWABuilder.

## Шаг 3. PWABuilder

1. https://pwabuilder.com → введите URL
2. Должны увидеть score ≥80 по Manifest и Service Worker
3. **Package For Stores → Android**
4. В диалоге заполните:
   - **Package ID:** `com.whofailed.game` (или другой уникальный — но запомните, его не сменить!)
   - **App name:** WHO Failed
   - **Launcher name:** WHO Failed
   - **App version:** 1.0.0
   - **Display mode:** Standalone
   - **Orientation:** Portrait
   - **Signing key:** "I'll provide my own" → или дайте PWABuilder сгенерировать новый (скачайте `signing.keystore` и сохраните пароль!)
5. Скачайте `.zip` с APK / AAB

## Если 500 повторяется

### Решение А — изменить Package ID

PWABuilder может выдавать 500 если кто-то уже собирал APK с этим Package ID. Попробуйте:
- `com.whofailed.game.v2`
- `com.yourname.whofailed`
- `app.whofailed`

### Решение B — обойти PWABuilder через bubblewrap (Google's official tool)

```bash
# Установить
npm i -g @bubblewrap/cli

# Инициализировать (вопросы — отвечайте по подсказкам)
bubblewrap init --manifest https://your-url.vercel.app/manifest.json

# Собрать APK
bubblewrap build
```

Это **тот же движок что использует PWABuilder под капотом** (TWA — Trusted Web Activity), но локально и без серверных 500 ошибок. Понадобится Java JDK 17+ и Android SDK.

### Решение C — Capacitor вместо TWA

TWA от Google требует постоянный HTTPS-доступ к вашему URL — приложение не работает оффлайн без сети для асет-валидации.

**Capacitor** упаковывает HTML внутрь APK как локальные файлы — игра работает оффлайн.

```bash
# В вашей папке who-failed-pwa
npm init -y
npm i -D @capacitor/cli @capacitor/core @capacitor/android
npx cap init "WHO Failed" "com.whofailed.game" --web-dir="."
npx cap add android
npx cap sync
npx cap open android
```

Откроется Android Studio → Build → Generate Signed Bundle/APK.

## Решение D — обратиться в поддержку PWABuilder

500 на их сервере — это их баг. Откройте issue:
https://github.com/pwa-builder/PWABuilder/issues

Приложите URL который пытались собрать и Package ID. Обычно отвечают за 1-2 дня.

## Что важно для Google Play

Независимо от инструмента сборки, для Play Store нужно:

- **Подписанный AAB** (не APK) — Google Play требует Android App Bundle с 2021 года
- **Keystore сохранён надёжно** — без него вы не сможете выпускать обновления (Google отказывается принимать ребилд с другим ключом)
- **Target SDK** — последний (на 2024-2025 это API 34 = Android 14)
- **Privacy Policy URL** — обязателен для всех приложений в Play
- **Скриншоты:** минимум 2 штуки, 1080×1920 или больше (можете сделать iPhone-Simulator или Android Emulator screenshot)
- **Feature graphic:** 1024×500 для store listing
- **Иконка 512×512** — уже есть
- **Content rating** — пройдите опросник в Play Console (вероятно Teen 12+ из-за пандемии)
- **Account fee:** $25 одноразово
