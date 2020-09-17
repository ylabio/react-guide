# Интеграция API

1. API - это запросы к серверу, обычно по принципу REST для получения данных и выполнения CRUD операций с ними.

2. Методы API используется в экшенах состояния, из контейнеров и других мест они не вызывается.

3. HTTP запросы выполняются библиотекой [`axios`](https://github.com/axios/axios).

4. Напрямую запросы не вызываются, все запросы к API описываются соответствующими функциями в директории `/api`.

5. Функции API сгруппированы по названиям ресурсов. 

6. Используются классы, чтобы наследовать типовые наборы функций API. Типовой набор - CRUD операции к ресурсу через
методы HTTP —  POST, GET, PUT, DELETE (иногда PATCH и HEAD).

7. Для интеграции с несколькими API (разные сервера) нужно создавать соответствующее количество отдельных директорий `/api*`.

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
