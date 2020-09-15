# Логика рендера

Для серверного рендера используется входной файл [`index.node.js`](https://github.com/ylabio/react-skeleton/blob/master/src/index.node.js). 
Его логика схожа с [`index.web.js`](https://github.com/ylabio/react-skeleton/blob/master/src/index.web.js). 
Но вместо монтирования приложения к DOM узлу вызывается рендер приложения в строку функцией [`renderToString()`](https://ru.reactjs.org/docs/react-dom-server.html#rendertostring).

*index.node.js*
```js
import { renderToString } from 'react-dom/server';
//..
const jsx = (
  <Provider store={store}>
    <Router history={navigation.history}>
      <App />
    </Router>
  </Provider>
);
const html = renderToString(jsx);
```
Перед рендером инициализируются объекты `api`, `navigation` (роутинг) и `store` (redux) - всё как для веб сборки. 
Особенности их исполнения на сервере определяются в конфигурации. Для api может использоваться другой базовый url, для навигации используется 
объект истории в памяти (вместо браузерного) и передаётся начальный url из адреса запроса в `initialEntries`, чтобы рендерить запрашиваемую страницу.

*index.node.js*
```js
import api from '@api';
import navigation from '@app/navigation';
import store from '@store';
//..
api.configure(config.api);
navigation.configure({ ...config.navigation, initialEntries: [workerData.url] }); // with request url
store.configure();
```

## Подготовка состояния

Чтобы рендер html был с полезным содержимым, его нужно выполнять после загрузки всех данных - полноценной инициализации 
состояния. 

> Если сейчас рендерить приложение, то получим страницу со статусом загрузки данных. Так как при первом рендере
состояние пустое. Компоненты успеют вызвать действия загрузки данных, но они асинхронные, и рендер в строку не будет ожидать
их выполнения. 

Все инициализации в контейнерах должны выполняться в хуке `useInit()`. Он основан на стандартном `useEffect()`, но работает
и при серверном рендере. Более того, этот хук на сервере добавляет промис асинхронной функции в массив `global.SSR.initPromises[]`.
Можно дождаться завершения всех промисов и выполнить рендер с уже полноценным состоянием и получить html с содержимым.

*index.node.js*
```js
(async () => {
  //..
  // Первичный рендер для инициализации состояния
  SSR.firstRender = true;
  renderToString(jsx);
  
  // Ждем все асинхронные функции, вызванные при первом рендере   
  await Promise.all(SSR.initPromises);

  // Итоговый рендер с инициализированным состоянием
  SSR.firstRender = false; // чтобы не работал хук useInit
  const html = renderToString(jsxExtractor);
})();
```

При повторном рендере хук `useInit()` на сервере уже не будет исполнять функцию. Он учитывает признак
`SSR.firstRender === true`. На клиенте используется тот же самый хук `useInit()`, но на клиенте, наоборот, только при 
первом рендере игнорируется вызов функции `useInit()`, если был активен серверный рендер (уже получен объект состояния).

> Могут быть подозрения в лишней нагрузки из-за двойного рендера. Но при первом рендере, обычно, нечего рендерить - 
> страница будет пустой. 99% временных задержек в сетевых запросах к АПИ. Оптимизировать нужно обращение к АПИ.
> Подготовительные операции через предварительный ренедер позволяют сохранить привычную архитектуру фронтенд приложения, 
> хук `useInit()` и так используется для упорядочивания логики приложения. Иной вариант потребует выносить роутинг из
> jsx разметки в отдельный конфиг, чтобы без рендера узнавать, какие компоненты будут рендериться. У этих компонент 
> должен бать статический методы инициализации. Страницы приложения вообще можно сделать обычными классами - не компонентами React. 
> Но это всё дальше от привычных практик разработки на React. С предварительным рендером нет опасений забыть подготовить 
> компонент для SSR. SSR просто работает :)

> Ещё один вопрос, достаточно ли двух рендеров? При втором рендере могут быть ещё какие-то инициализации... 
> Такая ситуация возможна, если не следовать рекомендациям по разделению логики приложения. Все инициализации должны
> выполняться в контейнере страницы в `/app`. Вообще не должно быть каскада загрузок/ожиданий, например, когда 
> вложенный контейнер рендерится после получения списка данных, и только потом запрашивает дополнительные данные. Он
> попросту не должен сам запрашивать данные.

## Метаданные, скрипты, стили

Последняя задача - вставить рендер приложения в шаблон html документа, а также прописать в шаблоне скрипты, стили и метаданные.
В качестве шаблона используется файла `/src/index.html`, в нём нет специальной разметки для шаблонизатора. Этот валидный
html файл, который используется и для сборки фронтенд. Для вставки в него данных используется функция `@utils/insert-text` 
для поиска тега и вставки строки перед или после него.

*index.node.js*
```js
import { parentPort, workerData } from 'worker_threads';
import insertText from '@utils/insert-text';
import template from './index.html';

//...
let out = template;
out = insertText.before(out, '<head>', baseTag + titleTag + metaTag);
out = insertText.after(out, '</head>', styleTags + linkTags + linkTags2);
out = insertText.before(out, '<div id="app">', html);
out = insertText.after(out, '</body>', scriptState + scriptTags);

// Передача результата в родительский процесс из Worker
parentPort.postMessage({ out, state, status: 200 });
```

Заголовок `<title>`, описание `<description>`, ключевые слова и другие мета теги `<meta>` определяются библиотекой [react-helmet](https://github.com/nfl/react-helmet#readme). 
Она используется в разметке jsx для динамической установки тегов в `<head>`. 

*index.node.js*
```js
import { Helmet } from 'react-helmet';
//...
// Метаданные рендера
const helmetData = Helmet.renderStatic();
const baseTag = `<base href="${config.navigation.basename}">`;
const titleTag = helmetData.title.toString();
const metaTag = helmetData.meta.toString();
const linkTags = helmetData.link.toString();
```

Стили и скрипты с учетом деления сборок на чанки и других особенностей сборки определяются библиотекой [loadable-components](https://loadable-components.com).
Её целевое назначение - динамический импорт компонент, но заодно предоставляет функционал для сбора статики при 
рендере в строку. При сборке приложения библиотека формируется файл с метаданными про сборку `/dist/node/loadable-stats.json`, 
руководствуясь этому файлу, библиотека узнает, какие скрипты и стили соответствуют текущему рендеру. Они вставляются в html.

*index.node.js*
```js
import { ChunkExtractor, ChunkExtractorManager } from '@loadable/server';
//...
const statsFile = path.resolve('./dist/node/loadable-stats.json');
const extractor = new ChunkExtractor({ statsFile });
const jsxExtractor = extractor.collectChunks(<ChunkExtractorManager extractor={extractor}>{jsx}</ChunkExtractorManager>);
//...
// Рендер jsxExtractor вместо jsx
const html = renderToString(jsxExtractor);
//...
// Скрипты, ссылки, стили с учётом параметров сборки
const scriptTags = extractor.getScriptTags();
const linkTags2 = extractor.getLinkTags();
const styleTags = extractor.getStyleTags();
```
