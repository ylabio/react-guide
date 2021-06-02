# Интеграция API

1. API - это запросы к серверу, обычно, по принципу REST для получения данных и выполнения операций с ними.

2. Функции API используются в экшенах состояния (store). Из контейнеров и других мест они не вызываются.

3. HTTP запросы выполняются библиотекой [`axios`](https://github.com/axios/axios).

4. Все запросы к API описываются соответствующими функциями в директории `/api`. Они используются в экшенах.

5. Функции API сгруппированы в поддиректории (файлы) по названиям ресурсов (моделей данных). 

6. Используются классы, чтобы наследовать типовые наборы функций API. Например, типовой набор представляет CRUD операции 
к ресурсу через методы HTTP — POST, GET, PUT, DELETE (иногда PATCH и HEAD). Могут быть другие типовые API.

7. Для интеграции с несколькими API (разных серверов) нужно создавать соответствующее количество отдельных директорий `/api*`.

*[`/src/api/users/index.js`](https://github.com/ylabio/react-skeleton/blob/master/src/api/users/index.js)*
```js
import params from '@utils/query-params';
import Common from '@api/common';

/**
 * API пользователей с наследованием CRUD методов и добавлением кастомных 
 */
export default class Users extends Common {
  /**
   * @param api {AxiosInstance} Экземпляр библиотеки axios
   * @param path {String} Путь в url по умолчанию
   */
  constructor(api, path = 'users') {
    super(api, path);
  }

  /**
   * Авторизация
   * @param login
   * @param password
   * @param remember
   * @param fields
   * @param other
   * @returns {Promise}
   */
  login({ login, password, remember = false, fields = '*', ...other }) {
    return this.http.post(
      `/api/v1/users/sign`,
      { login, password, remember },
      { params: params({ fields, ...other }) },
    );
  }

  /**
   * Выход
   * @returns {Promise}
   */
  logout() {
    return this.http.delete(`/api/v1/users/sign`);
  }
}
```

*Вызов API*
```js
import * as api from '@api';
//...
const response = await api.users.login(data);
```
