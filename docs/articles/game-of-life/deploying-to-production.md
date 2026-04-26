# Публикация веб-приложения «Игра жизни» в продакшен

Когда проект готов, следующий шаг — выложить его на продакшен-сервер вместо локального dev-сервера. С Rust и WebAssembly всё то же, что и в общем случае: файлы должны быть доступны из сети через HTTP-сервер!

Убедитесь, что wasm-модуль собран актуально: из каталога `wasm-game-of-life` выполните `wasm-pack`:

```
wasm-pack build
```

Затем соберите JavaScript и HTML через `webpack` в каталоге `wasm-game-of-life/www`:

```
./node_modules/.bin/webpack
```

В результате собирается всё веб-приложение — бинарник `.wasm`, JavaScript и HTML — и попадает в `wasm-game-of-life/www/dist`. Содержимое может выглядеть примерно так:

```
wasm-game-of-life/www/dist/
├── 0.bootstrap.js
├── 357c6c6c57e15cecdc07.module.wasm
├── bootstrap.js
└── index.html
```

Чтобы приложение открывалось с сервера, тот должен отдавать `.wasm` с правильным [MIME-типом](https://developer.mozilla.org/ru/docs/Web/HTTP/Basics_of_HTTP/MIME_types) — `application/wasm`.

> **Примечание:** настройка сервера зависит от ОС; лучше найти руководство под вашу систему и веб-сервер. Ниже предполагается что-то вроде Debian/Ubuntu или CentOS/Red Hat/Fedora.

Для nginx [добавьте `application/wasm wasm;` в `/etc/nginx/mime.types`](https://nginx.org/en/docs/http/ngx_http_core_module.html#types). Затем перезагрузите nginx: `sudo nginx -s reload`, чтобы подхватить изменения.

Для Apache [добавьте `AddType application/wasm .wasm` в корень конфигурации](https://httpd.apache.org/docs/2.4/mod/mod_mime.html#addtype) — часто это `/etc/apache2/apache2.conf` или `/etc/httpd.d/conf/httpd.conf`. Перезагрузите Apache: `sudo apachectl -k graceful`.

Наконец, можно залить содержимое `dist` на продакшен, например через SCP или SFTP-клиент. Если скопировать файлы в корень сайта (часто `/var/www/html` или `/var/www`), приложение станет доступно извне.
