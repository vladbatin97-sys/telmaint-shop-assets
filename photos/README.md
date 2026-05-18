# Product photos — массовая заливка

Каждый SKU имеет **свою** product-фотографию: товар в 3/4 ракурсе, прозрачный фон, «в полёте» (без чехлов, без рук, без мебели, без контекста).

Без manual photo используется generic Unsplash из `lib/catalog/mock.ts` — там iPhone в чехлах, MacBook с кофе, AirPods на руке. Это misleading клиента и НЕ соответствует брендбуку.

## Как добавить фото к товару (Маргарита)

### Шаг 1 — Скачать оригинал

Источники по приоритету (от лучшего к worst):

| Источник | Когда |
|---|---|
| **apple.com/ru** (Apple) | iPhone / MacBook / iPad / Watch / AirPods |
| **samsung.com/ru** (Samsung) | Galaxy / Buds / Watch |
| **mi.com/ru** или **xiaomi.ru** | Xiaomi / Redmi / POCO |
| **honor.ru / consumer.huawei.com** | Honor / Huawei |
| **dyson.ru** | Dyson |
| **bose.ru / sony.ru / jbl.com** | Headphones / Speakers |

Открываем страницу товара → правый клик на главное фото (3/4 ракурс) → «Сохранить изображение». Размер ≥1000×1000px.

### Шаг 2 — Удалить фон

Открываем https://www.remove.bg/ (бесплатный web-интерфейс, до 50 фото/месяц без регистрации):

1. Загружаем скачанное фото
2. Ждём 3-5 секунд
3. **Download** → формат PNG с прозрачностью

Альтернативы если remove.bg исчерпан:
- https://www.adobe.com/express/feature/image/remove-background (бесплатно, без лимитов)
- https://www.canva.com/photo-editor/ (BG-remover, бесплатно для аккаунтов)
- Photoshop / GIMP (вручную через волшебную палочку)

### Шаг 3 — Конвертация в WebP (опционально, рекомендуется)

WebP сжимает PNG в 2-3 раза без потери качества. Это ускоряет загрузку каталога.

Простой конвертер: https://squoosh.app/ (от Google)

1. Загружаем PNG
2. В правом меню выбираем **WebP**, качество 85%
3. Download

Если лень — оставляйте PNG, на сайт всё равно подхватится.

### Шаг 4 — Узнать SKU товара

Открыть товар на сайте, скопировать URL:

```
https://shop.telmaint.111-88-241-221.nip.io/product/iphone-17-pro-256-esim-blue--05531
                                                          ↑                          ↑↑↑↑↑
                                                          human-slug                 SKU
```

SKU = всё после последнего `--`. В примере: `05531`.

### Шаг 5 — Сохранить файл с правильным именем

Положить в `shop/public/photos/` под одним из имён:

**Вариант A — рекомендуемый (читаемый):**
```
iphone-17-pro-256-esim-blue--05531.webp
```

Скопируй из URL всё после `/product/` + добавь `.webp` (или `.png`/`.jpg`).

**Вариант B — короткий (только SKU):**
```
05531.webp
```

Скрипт `scan-photos.mjs` понимает оба варианта. Главное — SKU после `--` или всё имя без `--` совпадает с SKU товара.

### Шаг 6 — Передать Владу для commit + deploy

Маргарита **не делает git push сама**. Передаёт фото Владу (или присылает в чат) — Влад:

```bash
git add shop/public/photos/iphone-17-pro-256-esim-blue--05531.webp
git commit -m "feat(shop): photo для iPhone 17 Pro 256 Blue (#05531)"
git push
```

VPS-deploy запустит `prebuild` → `scan-photos.mjs` → перегенерит `lib/catalog/photos-manifest.json` → новое фото подхватится автоматически.

## Чек-лист принятия фото (Маргарита перед отправкой Владу)

- ✓ Фото **именно того** товара (точная модель, точный цвет, точная конфигурация)
- ✓ Ракурс **3/4** (не строго фронт, не профиль — диагональ ≈30-45°)
- ✓ Фон **прозрачный** (PNG/WebP с alpha-каналом). Открыть на чёрном фоне → видно товар без рамки
- ✓ Один экземпляр (не пара, не коллекция, не «семья продуктов»)
- ✓ Нет **чехла**, нет **рук**, нет **мебели**, нет **окружения**
- ✓ Нет watermark / логотипа источника
- ✓ Нет shadow или contact-shadow (товар «в полёте»)
- ✓ Размер файла **< 200 KB** (WebP/PNG, оптимизированный)
- ✓ Filename = `<slug-из-URL>.webp` или `<SKU>.webp`

## Технически (для Влада / разработчика)

- `shop/scripts/scan-photos.mjs` — Node script, сканирует `photos/*.{webp,png,jpg,jpeg}`, парсит SKU из filename, пишет `shop/lib/catalog/photos-manifest.json`.
- `package.json:predev` + `prebuild` — запускают scan автоматом перед `next dev` / `next build`.
- `shop/lib/catalog/mock.ts:getMockPhoto` — priority chain:
  1. `photos-manifest[sku]` → `/photos/<filename>` (manual product photo)
  2. Unsplash fallback из CURATED по category (legacy «iPhone в чехле»)
- `shop/components/catalog/ProductCard.tsx` — при `hasProductPhoto(sku)` рендерит `<Image src=manual />` с `object-contain` + padding (товар не обрезается).
- `shop/components/catalog/Mosaic.tsx` — при `hasProductPhoto` показывает single big photo вместо 4-grid (extras = Unsplash generic = misleading рядом с правильным фото).

## Текущее покрытие

Smoke-тест: `cat shop/lib/catalog/photos-manifest.json` показывает все привязанные фото.

Каталог 700+ SKU — массовая заливка по чек-листу выше. Приоритет: топ-50 самых продающихся моделей (iPhone 17 Pro Max / Pro / base, MacBook Pro 16 / Air, AirPods Pro, Apple Watch Series 11).
