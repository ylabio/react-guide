# Структура проекта

React приложение образуется иерархией компонентов с разным назначением. 
Одни компоненты отображают данные, другие управляют потоком данных, связыванием событий. 
Кроме компонент имеется состояние приложения, сервисы получения данных, вспомогательные утилиты и др. 

Для упорядочивания многообразия функций определена соответствующая структура:

```
├──/dist — сборки приложения после `npm run build` 
├──/src 
|    ├──/api — функции вызова rest api
|    |   ├──/common 
|    |   ├──/users
|    |   └──index.js — экземпляры axios библиотеки и функций апи
|    ├──/app  — страницы и разделы приложения, роутинг
|    |   ├──/home 
|    |   ├──/catalog
|    |   ├── index.js — корневой контейнер App
|    |   └── navigation.js — объект для навигации (history)
|    ├──/components — повторно используемые глупые компоненты
|    |   ├──/elements 
|    |   ├──/layouts
|    |   └──/menus
|    ├──/containers — повторно используемые фрагменты приложения — умные компоненты
|    |   ├──/article-list 
|    |   └──/main-menu
|    ├──/store — модули состояния на redux
|    |   ├──/articles
|    |   |   ├── state.js
|    |   |   ├── actions.js 
|    |   |   └── reducer.js
|    |   ├──/session 
|    |   |   └──...
|    |   ├──index.js — объект хранилища
|    |   └──reducers.js — реэкспорт редьюсеров из всех модулей состояний
|    ├──/theme — общие файлы стилей
|    ├──/utils — функции, созданные под проект, в том числе утилиты скелетона
|    ├── config.js — настройки проекта
|    ├── index.html — единственный шаблон с корневыми тегами документа 
|    ├── index.node.js — главный файл приложения для NodeJS при SSR
|    └── index.web.js — главный файл приложения для браузера
├── ...
├── package.json
├── server.js — сервер рендера и запуск node сборки
└── webpack.config.js — опции сброки и дев режима
```

Главный исполняемый файл фронтенд приложения - `/src/index.web.js`. 
Для исполнения на сервере — `/src/index.node.js`. 
У них общая задача — запустить рендер react компонентов, но отличия в инициализации приложения
из-за разного окружения.

- [/app](/docs/develop/structure/app.md)
- [/containers](/docs/develop/structure/containers.md)
- [/components](/docs/develop/structure/components.md)
- [/store](/docs/develop/structure/store.md)
- [/api](/docs/develop/structure/api.md)
- [/utils](/docs/develop/structure/utils.md)
- [/theme](/docs/develop/structure/theme.md)