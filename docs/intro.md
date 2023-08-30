---
sidebar_position: 1
---
# API для предзагрузки страниц
**Версия браузера**: `1.0.0`


## Введение
Механизм предназначен для ускорения работы веб интерфейсов с помощью предзагрузки страниц. Это улучшает восприятие переходов между страницами конечным пользователем. Также есть возможность сохранения и передачи необходимых данных (например, для аутентификации) в предзагружаемые страницы. 

Механизм реализуется посредством вызова метода `postMessage` следующих `messageHandlers`:
- `loonaStorage.set`
- `loonaStorage.get`
- `preloadPages`
- `preloadRedirect`

И передачи сообщения (объект) в формате `JSON string`. 

:::tip Обратите внимание
Объект должен соответствовать протоколу [JSON-RPC](https://www.jsonrpc.org/specification) версии 2.0.
:::

## Методы
### loonaStorage.set
Сохраняет `<значение>` для `<ключа>` в хранилище браузера `loonaStorage`.

`<значение>` и `<ключ>` принимают значение типа string.


```javascript
window
    .webkit
    .messageHandlers
    .loonaStorage
    .postMessage(
        JSON.stringify({
            "jsonrpc": "2.0",
            "method": "set",
            "params": {"key":   "<ключ>",
                    "value": "<значение>"},
            "id" : 1,
        })
    );
```

### loonaStorage.get
Запрашивает значение по `<ключу>` из хранилища браузера `loonaStorage`.

```javascript
window
    .webkit
    .messageHandlers
    .loonaStorage
    .postMessage(
        JSON.stringify({
            "jsonrpc" : "2.0",
            "method" : "get",
            "params" : {"key" : "<ключ>"},
            "id" : 2,
        })
    );
```
Для получения в браузере запрошенного значения по ключу нужно повесить слушатель(EventListener) по событию `message`. 

```javascript
window.addEventListener('message', handlerExample);
```
handlerExample

!описать пример handlerExample

!Ответ от браузера приходит в формате JSON-RPC. 


### preloadPages
Регистрирует массив ссылок для перехода на другие сайты.

В параметре `urls` передаётся массив ссылок. Ссылки должны быть абсолютными.

```javascript
window
    .webkit
    .messageHandlers
    .preloadPages
    .postMessage(
        JSON.stringify({
            "jsonrpc" : "2.0",
            "method" : "preload",
            "params" : {"urls" : ["<ссылка 1>","<ссылка 2>","<ссылка n>"]},
                "id" : 3,
        })
    );
```
### preloadRedirect
Активирует переход по ранее зарегистрированной предзагруженной ссылке.

:::tip
Если ссылка не была ранее зарегистрирована и предзагружена с помощью метода preloadPages, то выполнится стандартная загрузка.
:::

```javascript
window
    .webkit
    .messageHandlers
    .preloadRedirect
    .postMessage(
        JSON.stringify({ 
            "jsonrpc" : "2.0",
            "method" : "preload",
            "params" : {"url" : "<ссылка для перехода>"},
            "id" : 4,
        })
    );
```


## Примеры использования

### Предзагрузка страницы без аутентификации
Задача:
Для нашего сайта, мы хотим реализовать в браузере предзагруженную страницу с погодой в Санкт-Петербурге.

Решение:

1) Сохраняем две ссылки.
```javascript
const URL_FOR_HAND_OVER_NUMBER_ONE = 'https://dzen.ru/';
const URL_FOR_HAND_OVER_NUMBER_TWO = 'https://dzen.ru/pogoda/saint-petersburg?lat=59.938951&lon=30.315635';
```

2) Отправляем браузеру информацию о том, какие ссылки нужно предзагрузить. 
```javascript
window
      .webkit
      .messageHandlers
      .preloadPages
      .postMessage(
          JSON.stringify({
              "jsonrpc" : "2.0",
              "method" : "preload",
              "params" : {"urls" : [URL_FOR_HAND_OVER_NUMBER_ONE, URL_FOR_HAND_OVER_NUMBER_TWO]},
                  "id" : 3,
          })
      );
```

3) Через 5 секунд, отправляем браузеру информацию на какую ссылку надо перейти(`https://dzen.ru/pogoda/saint-petersburg?lat=59.938951&lon=30.315635`).
```javascript
setTimeout(() => { 
                window
                    .webkit
                    .messageHandlers
                    .preloadRedirect
                    .postMessage(
                        JSON.stringify({
                            "jsonrpc" : "2.0",
                            "method" : "get",
                            "params" : {"url" : URL_FOR_HAND_OVER_NUMBER_TWO},
                            "id" : 4,
                        })
                    );
            }, POST_MESSAGE_DELAY)
```

4) При клике пользователя по <ins>ссылке на Дзен-Погода в Санкт-Петербурге</ins>, браузер отобразит предзагруженную страницу с погодой.  
Теперь можно визуально сравнить, что при клике и переходе на <ins>ссылку на Дзен-Погода в Санкт-Петербурге</ins> страница отображается, с предзагруженными данными, мгновенно.
При клике на <ins>ссылку на Дзен</ins> страница отображается без предзагруженных данных.

Данный пример подробнее, с отображением двух ссылок на нашей странице. 
Инструменты: React.

```javascript
import { useEffect} from "react";

import "../styles/title.css";

const URL_FOR_HAND_OVER_NUMBER_ONE = 'https://dzen.ru/';
const URL_FOR_HAND_OVER_NUMBER_TWO = 'https://dzen.ru/pogoda/saint-petersburg?lat=59.938951&lon=30.315635';

const POST_MESSAGE_DELAY = 5000;

export default function OursComponent(){

    useEffect( () => {
        if (window.webkit?.messageHandlers?.preloadPages) {
            window
                .webkit
                .messageHandlers
                .preloadPages
                .postMessage(
                    JSON.stringify({
                        "jsonrpc" : "2.0",
                        "method" : "preload",
                        "params" : {"urls" : [URL_FOR_HAND_OVER_NUMBER_ONE, URL_FOR_HAND_OVER_NUMBER_TWO]},
                            "id" : 3,
                    })
                );

            setTimeout(() => { 
                window
                    .webkit
                    .messageHandlers
                    .preloadRedirect
                    .postMessage(
                        JSON.stringify({
                            "jsonrpc" : "2.0",
                            "method" : "get",
                            "params" : {"url" : URL_FOR_HAND_OVER_NUMBER_ONE},
                            "id" : 4,
                        })
                    );
            }, POST_MESSAGE_DELAY)
        }
    },[])

    return (
        <>
            <a
                href='https://dzen.ru/'
            >
                ссылка на Дзен
            </a>
            <br/>
            <a
                href='https://dzen.ru/pogoda/saint-petersburg?lat=59.938951&lon=30.315635'
            >
                ссылка на Дзен-Погода в Санкт-Петербурге
            </a>
        </>
    )
}
```

### Предзагрузка страницы с аутентификацией
Задача: Передать рефреш токен из одной нашей страницы на другую.

Решение:

Отправляем браузеру сохранить в хранилище  `loonaStorage` refresh токен.
```javascript
window
    .webkit
    .messageHandlers
    .loonaStorage
    .postMessage(
        JSON.stringify({
            "jsonrpc": "2.0",
            "method": "set",
            "params": {"key":   "token",
                    "value": "yourstoken"},
            "id" : 1,
        })
    );
```

Получаем refresh токен по ключу `token` из хранилища браузера `loonaStorage`.
```javascript
window
    .webkit
    .messageHandlers
    .loonaStorage
    .postMessage(
        JSON.stringify({
            "jsonrpc" : "2.0",
            "method" : "get",
            "params" : {"key" : "token"},
            "id" : 2,
        })
    );
```

Отправляем браузеру информацию о том, какие ссылки нужно предзагрузить. 
```javascript
window
      .webkit
      .messageHandlers
      .preloadPages
      .postMessage(
          JSON.stringify({
              "jsonrpc" : "2.0",
              "method" : "preload",
              "params" : {"urls" : [URL_FOR_HAND_OVER_NUMBER_ONE, URL_FOR_HAND_OVER_NUMBER_TWO]},
                  "id" : 3,
          })
      );
```
Отправляем браузеру информацию на какую ссылку надо перейти(``).
```javascript
 
window
    .webkit
    .messageHandlers
    .preloadRedirect
    .postMessage(
        JSON.stringify({
            "jsonrpc" : "2.0",
            "method" : "get",
            "params" : {"url" : URL_FOR_HAND_OVER_NUMBER_TWO},
            "id" : 4,
        })
    );
```

Данный пример подробнее, с отображением двух ссылок на нашей странице. 
Инструменты: React.