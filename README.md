<div align="center">

[![Core](https://img.shields.io/badge/Core-free--turn--proxy-blue?logo=github&logoColor=white)](https://github.com/purkalolka/free-turn-proxy)
![Android](https://img.shields.io/badge/Android-6.0%2B-3DDC84?logo=android&logoColor=white)
![Kotlin](https://img.shields.io/badge/Kotlin-Compose-7F52FF?logo=kotlin&logoColor=white)
![Material 3](https://img.shields.io/badge/Material-3-757575?logo=materialdesign&logoColor=white)
![License](https://img.shields.io/badge/license-GPL--3.0-blue)
</div>

![Banner](assets/banner.jpg)

> **Disclaimer.** Проект предназначен **исключительно для образовательных и исследовательских целей.**

> **Важно:** При обновлении до версии 3.0.0 все настройки будут сброшены.

## Возможности

- **Добавление нескольких серверов**
- **Клонирование конфигурации серверов**
- **Быстрая установка на VPS**
- **Возможность делиться конфигами**
- **Режим работы прокси / VPN** (WireGuard)
- **UDP-релей до TURN** - бэкенд на сервере только UDP (WireGuard / AmneziaWG)
- **Бэкапы**
- **Раздельное туннелирование**

## Улучшения и исправления в ядре ([free-turn-proxy v3.4.1](https://github.com/purkalolka/free-turn-proxy))

В приложение интегрировано обновленное ядро `v341-vkcalls` с критическими исправлениями безопасности и стабильности:

- **Авторизация VK Calls без капчи:**
  - Использование официального мобильного идентификатора (`client_id=8093730`) для вызовов `auth.getAnonymToken` и `messages.getAnonymCallToken`, что полностью исключает появление капчи при получении TURN-токенов.
- **DTLS Certificate Pinning (защита от MITM):**
  - Поддержка проверки и привязки отпечатка сертификата (SHA-256 fingerprint: `sha256:...`) на стороне DTLS-клиента. Сервер сохраняет постоянный сертификат между перезапусками, предотвращая подмену TLS-сессии.
- **Защита от атак повтора (Anti-Replay Window):**
  - Реализован 128-битный скользящий фильтр повторов (RFC 6479 / RFC 4303) в обработке инкапсулированных пакетов RTP Opus, отсекающий повторно отправленные перехваченные пакеты.
- **Безопасные права доступа:**
  - Права на хранилище сохраненных учетных данных и конфигураций ограничены до `0600` (`-rw-------`).
- **Отказоустойчивое развертывание сервера:**
  - В скриптах `server-control` внедрены повторные попытки (retry) с контролем зависаний потока при скачивании бинарников, сверка SHA-256 по `checksums.txt` и надежная обработка блокировок пакетных менеджеров.

## Требования

- **Android 7.0+** (API 24)
- **Архитектура процессора:** `arm64-v8a` или `armeabi-v7a`
- **VPS**
- **Ссылка на звонок**

## Благодарности

- **[@Moroka8](https://github.com/Moroka8)** - форк ядра [vk-turn-proxy](https://github.com/Moroka8/vk-turn-proxy)
- **[@alexmac6574](https://github.com/alexmac6574)** - форк ядра [vk-turn-proxy](https://github.com/alexmac6574/vk-turn-proxy)
- **[@cacggghp](https://github.com/cacggghp)** - оригинальное [vk-turn-proxy](https://github.com/cacggghp/vk-turn-proxy)
- **[@MYSOREZ](https://github.com/MYSOREZ)** - оригинальный Android-клиент [vk-turn-proxy-android](https://github.com/MYSOREZ/vk-turn-proxy-android)
