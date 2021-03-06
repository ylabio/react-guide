# `/app`

Скелет всего приложения. Вложенность `/app` соответствуют карте (роутингу) сайта.

Каждый компонент в `/app` — это страница или раздел сайта. 
Компонентами '/app' определяется роутинг и основная логика приложения. 
Например, запрос первичных данных с сервера АПИ, обработка параметров url, контроль доступа и прочее.

В `/app` не допускается вёрстка (стилизация), вместо неё используется разметка 
на “глупых” компонентах из `/components` или из сторонних библиотек.

Логику `/app` можно декомпозировать на повторно используемые логические части. 
На “умные” компоненты — контейнеры `/containers`. Тем самым упрощается логика в `/app`. 

```
├──/app
   ├──/home - главная страница
   ├──/catalog - страница каталога товаров
   ├── index.js — корневой контейнер App
   └── navigation.js — объект для навигации (history)
```

📖 [Роутинг и навигация](/docs/check/router.md)