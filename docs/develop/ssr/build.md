# Сборка и запуск SSR

Для серверного рендера создаётся своя сборка. Используется общий файл настроек [`webpack.config.js`](https://github.com/ylabio/react-skeleton/blob/master/webpack.config.js). 
В зависимости от назначения сборки меняется параметр `libraryTarget` на `'commonjs2'` и входной файл приложения в `entry` на `'src/node.node.js'`. 
Опции модулей, траспиляции и прочего одинаковые. 

*[`webpack.config.js`](https://github.com/ylabio/react-skeleton/blob/master/webpack.config.js)*
```js
let config = {
  name: target, // "web" или "node" передаётся в параметрах запуска сборки
  target: target,
  mode: process.env.NODE_ENV, // https://webpack.js.org/configuration/mode/
  entry: [`index.${target}.js`], //index.node.js или index.web.js
  output: {
    path: path.join(__dirname, 'dist', target), // В разные папки сборки
    libraryTarget: isNode ? 'commonjs2' : undefined,  // Для Node.js используется модули в CommonJS
    //...
  },
  //...
}
```

Чтобы в логике приложения учитывать режим запуска, в сборке определены константы:

- `TARGET`: string - oneOf('web', 'node')
- `IS_WEB`: boolean // признак, приложение для браузера
- `IS_NODE`: boolean // признак, приложение для сервера

*Пример в коде приложения*
```js
if (process.env.IS_NODE) {
  // приложение запущено на Node.js
}
if (process.env.IS_WEB) {
  // приложение запущено в браузере
}
````

Для запуска приложения с серверным рендером используется команда: 
```
npm run start:ssr 
```

Запуск SSR в режиме продакшена, т.е. запускается сборка. Поэтому автоматически будут созданы новые сборки для Node.js и web. 

Отдельно сборка выполняется командой `npm run build:ssr`. Нужно не забывать делать сборку и для веба `npm run build`. 
Можно одной командой все сразу: 
```
npm run build:all
```

На продакшен сервере пересобирать приложение не надо, остаётся только запустить. Поэтому npm команда start не подойдет (так как делает сборку),
лучше напрямую запустить Node.js с файлом [/server.js](https://github.com/ylabio/react-skeleton/blob/master/server.js). Этот файл не попадает в сборку, не является частью фронтенд приложения, 
а содержит логику http сервера и сам запускает сборку фронта под Node.js для обработки запросов.
```
node ./server.js
```

Про команды [сборки и запуска SSR](/docs/installation/build.md).

В сборку попадают все используемые пакеты из `/node_modules`. На сервере не надо устанавливать npm пакеты. 
Более того, пакеты для фронта часто оказываются нерабочими в среде node.js. Сборка всех пакетов через webpack решает эту проблему. 
При желании из сборки можно исключить кроссплатформенные пакеты, указав их названия в опции вебпака `externals`. 
Node.js будет искать тогда пакет в директории `/node_modules`. 

*[`webpack.config.js`](https://github.com/ylabio/react-skeleton/blob/master/webpack.config.js)*
```js
if (isNode) {
   config.externals = ['react-helmet', 'moment'];
}
```
   
> В теории приложение на сервере можно было бы запускать без сборки. 
Синтаксические особенности javascript, которые не поддерживает Node.js (их немного), можно активировать через транспиляцию 
в runtime режиме библиотеками [babel-runtime](https://babeljs.io/docs/en/babel-runtime) 
или [core-js](https://github.com/zloirock/core-js). 
Куда сложнее проблема импорта не js файлов в Node.js. Webpack позволяет импортировать любые файлы, описывав способ 
импорта в опциях `module.rules`. Для Node.js нужно глобально переопределить функцию require() 
([пример](https://stackoverflow.com/questions/19903398/node-js-customize-require-function-globally)) 
и на все кастомные типы файлов реализовывать особый импорт. Например, импорт png файла - это генерация относительного url или base64 ссылки.
Импорт стилей просто игнорируется, либо формируется объект с хэшами всех ccs-классов. Есть библиотеки, которые анализирую, 
как вебпак импортировал для веба файл и повторяют логику для ноды. Варианты решений без сборки под Node.js постоянно натыкались 
на грабли совместимости, корректности путей, лишних оберток. 
Надежное и простое решение - делать отдельную сборку для Node.js.

