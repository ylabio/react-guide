# `/containers`

Самостоятельные части страниц, вынесенные в отдельные “умные” компоненты из app в целях повторного 
использования или снижения сложности кода в app. 
Без вёрстки, только логика как в app, но без роутинга (иначе нарушится принцип соответствия app карте сайта). 
Пример: боковая панель, повторно используемая на каждой странице сайта. По паттерну MVC компоненты-контейнеры тоже являются контроллерами. Можно также считать, что контейнеры представляющие страницу или раздел сайта вынесены в app.

Контейнеры дополняют функциональность приложения в app, но не определяют её основу. 
Например, получение начальных данных должно выполняться на уровне app, а в containers этими данными 
можно оперировать — брать и изменять. Из-за отсутствия какого-либо контейнера не должна нарушаться 
логика страницы или других контейнеров.

- [Требования к контейнерам](/docs/check/container.md)

