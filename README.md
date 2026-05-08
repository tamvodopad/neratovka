# neratovka.ru

Сайт села Нератовка (Заинский район, Республика Татарстан).

Сейчас содержит одну страницу — **Книгу Памяти** уроженцев села, участников Великой Отечественной войны.

## Структура

```
html/
  index.html   — placeholder (пока пусто, noindex)
  ww2.html     — Книга Памяти 1941–1945, 110 уроженцев
docker-compose.yml — деплой через nginx + reverse-proxy
nginx.conf         — конфиг внутреннего nginx
```

## Локальная разработка

```bash
cd html && python3 -m http.server 8080
# открыть http://localhost:8080/ww2.html
```

## Деплой через Portainer

### 1. Проверить какой reverse-proxy установлен на сервере

SSH на `109.120.138.8` и:

```bash
docker ps --format '{{.Names}} | {{.Image}}'
```

Найти контейнер из:
- `nginx-proxy` или `jwilder/nginx-proxy` → **Option A** в docker-compose.yml (env уже включён)
- `traefik` → **Option B** (раскомментировать labels, закомментировать environment)
- `caddy` → конфигурируется в Caddyfile хоста, не в docker-compose

Также проверить имя сети:
```bash
docker network ls | grep -E 'proxy|traefik'
```

Если сеть называется не `nginx_proxy` — поправить в `docker-compose.yml` (раздел `networks:`).

### 2. Создать Stack в Portainer

Два варианта:

**a) Через Git** (рекомендую — авто-деплой при пуше в main):
1. Portainer → Stacks → Add stack
2. Build method: **Repository**
3. Repository URL: `https://github.com/tamvodopad/neratovka`
4. Compose path: `docker-compose.yml`
5. Включить **Automatic updates** + webhook (Portainer вызовет re-deploy после `git push`)

**b) Через Web editor:**
1. Portainer → Stacks → Add stack → Web editor
2. Вставить содержимое `docker-compose.yml`
3. Deploy

### 3. DNS

В DNS-зоне `neratovka.ru` создать A-записи:

```
neratovka.ru.         A  109.120.138.8
www.neratovka.ru.     A  109.120.138.8
```

После DNS-распространения и поднятия контейнера сайт будет доступен на:
- `https://neratovka.ru/ww2.html` — Книга Памяти
- `https://neratovka.ru/` — placeholder (пока пусто)

## Что внутри ww2.html

Полностью самостоятельный HTML-файл. Никаких внешних зависимостей кроме Google Fonts (PT Serif, Old Standard TT). Все 110 уроженцев + статистика + графики + интерактивный список встроены в один файл (~80 КБ).

Источник данных — открытый архив [«Память народа»](https://pamyat-naroda.ru) Министерства обороны РФ.
