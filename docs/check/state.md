# Состояние и действия

1. Используется библиотека [`redux`](https://redux.js.org).
    - Без применения redux-thunk и прочих оберток.
    - Объект хранилища `store` доступен отовсюду через `import store form @src/store`.

2. Всё состояние приложения декомпозируется на модули. В модуле определяются экшены, начальное состояние и редьюсер по установке состояния.

```
├──/store — модули состояния на redux
   ├──/articles
   |   ├── state.js
   |   ├── actions.js 
   |   └── reducer.js
   ├──/session 
   |   └──...
   ├──index.js — объект хранилища
   └──reducers.js — реэкспорт редьюсеров из всех модулей состояний

```
3. **Типы экшенов** и **начальное состояние** описывается в отдельном файле [`{module}/state.js`](https://github.com/ylabio/react-skeleton/blob/master/src/store/articles/state.js).

4. **Экшены** — функции для выполнения различной логики и последующей смены состояния.
    - Вызов экшена — это обычный вызов функции, не надо оборачивать его в `dispatch()`.
    - Экшен напрямую обращается к объекту хранилища, вызывая `store.dispatch({type, payload})` или `store.getState()`
    - Экшены описываются в файле [`{module}/actions.js`](https://github.com/ylabio/react-skeleton/blob/master/src/store/articles/actions.js) 
      в виде методов объекта. 
    - Можно применять классы для описание экшенов, но экспортировать экземпляр класса. Классы позволяют наследовать типовые наборы экшенов.
    - Экшены напрямую импортируется в [контейнерах](/docs/check/container.md) для вызовов.

5. **Редьюсеры** описываются в файле [`{module}/reducer.js`](https://github.com/ylabio/react-skeleton/blob/master/src/store/articles/reducer.js)
    - Описываются через функцию-обертку, которой передаётся начальное состояние, типы экшенов и объект с кастомными редьюсерами. 
    - Обычно логика всех редьюсеров проста и идентична —  присвоить новый объект состояния. 
    - Используется библиотека [`merge-change`](https://www.npmjs.com/package/merge-change) для слияния нового состояния со старым.
    - Файл редьюсера нужно реэкспортировать в [`@src/store/reducers.js`](https://github.com/ylabio/react-skeleton/blob/master/src/store/reducers.js)

*Экшены формы входа [`src\store\form-login\actions.js`](https://github.com/ylabio/react-skeleton/blob/master/src/store/form-login/actions.js)*
```js
import store from '@src/store';
import * as api from '@src/api';
import initState, { types } from './state.js';

export default {
  /**
   * Изменение полей формы
   * @param data
   */
  change: data => {
    store.dispatch({type: types.SET, payload: { data }});
  },

  /**
   * Отправка формы в АПИ
   * @param data
   * @returns {Promise<*>}
   */
  submit: async data => {
    store.dispatch({ type: types.SET, payload: { wait: true, errors: null } });
    try {
      const response = await api.users.login(data);
      //...
      // сброс данных в форме
      store.dispatch({ type: types.SET, payload: initState });
      return result;
    } catch (e) {
      //..
    }
  },
};
```

*Тип экшенов и начальное состояние [`src\store\form-login\state.js`](https://github.com/ylabio/react-skeleton/blob/master/src/store/form-login/state.js)*
```js
export const types = {
  SET: Symbol('SET'),
};

export default {
  data: {
    login: '',
    password: '',
  },
  wait: false,
  errors: null,
};
```

*Редьюсер [`src\store\form-login\reducer.js`](https://github.com/ylabio/react-skeleton/blob/master/src/store/form-login/reducer.js)*
```js
import reducer from '@src/utils/reducer';
import mc from 'merge-change';
import initState, { types } from './state.js';

export default reducer(initState, {
  [types.SET]: (state, action) => {
    return mc.update(state, action.payload);
  },
});
```

*Пример вызова экшенов в форме авторизации [`/src/app/login/index.js`](https://github.com/ylabio/react-skeleton/blob/master/src/app/login/index.js)*
```js
import formLogin from '@src/store/form-login/actions';

function Login(props) {
  //...
  const callbacks = {
    onChangeForm: useCallback(async data => {
      // Экшен по установке данных из формы в общий стейт
      await formLogin.change(data);
    }, []),
    onSubmitForm: useCallback(async data => {
      // Отправка данных формы
      await formLogin.submit(data);
      //..
    }, []),
  };
  // ...
```
