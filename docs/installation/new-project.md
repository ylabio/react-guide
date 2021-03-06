# Новый проект

## Создание

Проект создается форком или клонированием репозитория **[react-skeleton](https://github.com/ylabio/react-skeleton.git)** 
для дальнейшего свободно модифицировать кода под особености проекта. 
В случае клонирования потребуется добавить `remote-url` на новый внешний репозиторий, например в gitlab.

1. Клонировать [react-skeleton](https://github.com/ylabio/react-skeleton.git): 
    ```
    git clone https://github.com/ylabio/react-skeleton.git
    git checkout master
    ```
2. Удалить текущий origin на skeleton:
    ```
    git remote rm origin
    ```
3. Добавить адрес на новый внешний репозиторий и назвать его origin: 
    ```
    git remote add origin https://github.com/VladimirShestakov/doc-site.git 
    ```
    Внешний репозиторий нужно предварительно создать через интерфейс хостинга. Обязательно пустым без добавления файлов readme и прочих.
4. Запушить master ветку в новый внешний репозиторий и связать локальную ветку master с той же веткой во внешнем репозитории, чтобы после не указывать refspec при пушах: 
    ```
    git push origin refs/heads/master:refs/heads/master
    git branch --set-upstream-to=origin/master master
    ```

## Инициализация
В директории проекта выполнить команду для установки npm пакетов.
```
npm install
```

Все зависимые пакеты будут загружены в директорию node_modules. Можно добавлять дополнительные пакеты командой 
```
npm install --save <package-name>
```

## Запуск для разработки

В режиме разработки (development)  приложение запускается командой:
```
npm start
```

Приложение по умолчанию доступно по адресу `http://localhost:8031`. Порт меняется в файле конфигурации 
[`src/config.js`](https://github.com/ylabio/react-skeleton/blob/master/src/config.js)) и может отличаться от указанного.

*[`/src/config.js`](https://github.com/ylabio/react-skeleton/blob/master/src/config.js)*
```javascript
let config = {
 dev: {
   port: 8031,
 },
 // ...
}
```
В режиме разработки используется локальный webpack http сервер для отслеживания изменения в коде и последующего "горячего" обновления приложения в браузера. 
