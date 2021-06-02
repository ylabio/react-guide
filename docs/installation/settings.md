# Настройка

## Опции сборки

Параметры сборки определяются в файле [`webpack.config.js`](https://github.com/ylabio/react-skeleton/blob/master/webpack.config.js). 
По умолчанию учтены режимы разработки/продакшена, сборки для серверного рендера, импорта файлов с jsx, less, картинок, шрифтов, прокси для апи (обхода CORS).

## Опции приложения

Конфигурация самого приложения определяется в файле [`src/config.js`](https://github.com/ylabio/react-skeleton/blob/master/src/config.js). 
По умолчанию определены параметры АПИ, роутинга. 
Здесь же параметры сервера разработки и рендера. В `src/config.js` можно добавлять свои параметры приложения.

## Опции развертывания

На сервер заливается директория со сборками `/dist`. Сборку можно сделать и на сервере, тогда на сервер заливаются исходники проекта,
обычно, через git. 

В качестве http сервера рекомендуется использовать Nginx. Задача Nginx отдавать статичные файлы из 
`./dist/web`, а если url не на существующий файл, то отдавать файл `./dist/web/index.html`.

*Пример настройки Nginx*
```
server {
    listen 80;
    server_name react-skeleton.com;
    location / {
        root /home/user/react-skeleton/dist/web;
        try_files $uri /index.html;
    }
}
```

Если планируется использовать серверный рендер (SSR), то потребуется запустить приложение на node.js и вместо отдачи 
`./dist/web/index.html` проксировать запросы в серверное приложение. Для этого в настройках Nginx опции
проксирования, если запрос на несуществующий файл.

*Пример настройки Nginx с SSR*
```
server {
    listen 80;
    server_name react-skeleton.com;
    location / {
        root /home/user/react-skeleton/dist/web;
        try_files $uri @ssr;
    }

    location @ssr {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Frame-Options   SAMEORIGIN;
        proxy_pass http://127.0.0.1:8132;
    }
}
```

Для постоянной работы приложения рендера воспользуйтесь менеджером процессов [pm2](https://pm2.keymetrics.io/docs/usage/pm2-doc-single-page/). Запуск из корневой директории приложения,
где кроме папки со сборками нужен файл [`/process.json`](https://github.com/ylabio/react-skeleton/blob/master/process.json) и [`/server.js`](https://github.com/ylabio/react-skeleton/blob/master/server.js)
```
pm2 start process.json 
```

## Алиасы путей

Вместо указания относительных путей можно применить алиас на src директорию.

*Пример импорта через алиас*
```js
import Button from '@src/components/button'; // вместо '../../../components/button'
```
