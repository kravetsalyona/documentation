---
sidebar_position: 2
---
# API для предзагрузки страниц актуальная версия
**Версия браузера**: `2.0.0`


## Введение
Механизм предназначен для ускорения работы веб интерфейсов с помощью предзагрузки страниц. Это улучшает восприятие переходов между страницами конечным пользователем. Также есть возможность сохранения и передачи необходимых данных (например, для аутентификации) в предзагружаемые страницы. 

Механизм реализуется посредством вызова метода `postMessage` у`GeneralHandler`. Название handler мы передаём внутри сообщения:
- `loonaStorage get`
- `loonaStorage set`
- `preloadedPages preload`
- `preloadedPages redirect`

И передачи сообщения (объект) в формате `JSON string`. 

:::tip Обратите внимание
Объект должен соответствовать протоколу [JSON-RPC](https://www.jsonrpc.org/specification) версии 2.0.
:::

## Методы
### loonaStorage set
Сохраняет `<значение>` для `<ключа>` в хранилище браузера `loonaStorage`.

`<значение>` и `<ключ>` принимают значение типа string.


```javascript
window
    .webkit
    .messageHandlers
    .GeneralHandler
    .postMessage(
        JSON.stringify({
            "jsonrpc": "2.0",
            "handler": "loonaStorage",
            "body": {
                    "method": "set",
                    "params": {
                                "key":   "<ключ>",
                                "value": "<значение>"
                    },
            },
            "id" : 1,
        })
    );
```

### loonaStorage get
Запрашивает значение по `<ключу>` из хранилища браузера `loonaStorage`.

```javascript
window
    .webkit
    .messageHandlers
    .GeneralHandler
    .postMessage(
        JSON.stringify({
            "jsonrpc" : "2.0",
            "handler": "loonaStorage",
            "body": {
                    "method" : "get",
                    "params" : {"key" : "<ключ>"},
            },
            "id" : 2,
        })
    );
```
Для получения в браузере запрошенного значения по ключу нужно повесить слушатель(EventListener) по событию `message`. 

```javascript
window.addEventListener('message', handlerExample);
```
```javascript
const handlerExample = (event) => {
      /*  Описываем поход с данными(токеном) из loonaStorage на наш сервер для аутентификации и авторизации.
          Ответ(объект) от сервера приходит в формате `JSON string`. Объект в формате JSON-RPC.
      */
    }
```


### preloadedPages preload
Регистрирует массив ссылок для перехода на другие сайты.

В параметре `urls` передаётся массив ссылок. Ссылки должны быть абсолютными.

```javascript
window
    .webkit
    .messageHandlers
    .GeneralHandler
    .postMessage(
        JSON.stringify({
            "jsonrpc" : "2.0",
            "handler": "preloadedPages",
            "body": {
                    "method" : "preload",
                    "params" : {
                                "urls" : ["<ссылка 1>","<ссылка 2>","<ссылка n>"]
                    },
            },
            "id" : 3,
        })
    );
```
### preloadedPages redirect
Активирует переход по ранее зарегистрированной предзагруженной ссылке.

:::tip
Если ссылка не была ранее зарегистрирована и предзагружена с помощью метода preloadedPages, то выполнится стандартная загрузка.
:::

```javascript
window
    .webkit
    .messageHandlers
    .GeneralHandler
    .postMessage(
        JSON.stringify({ 
            "jsonrpc" : "2.0",
            "handler": "preloadedPages",
            "body": {
                    "method" : "redirect",
                    "params" : {"url" : "<ссылка для перехода>"
                    },
            },
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
      .GeneralHandler
      .postMessage(
        JSON.stringify({
            "jsonrpc" : "2.0",
            "handler": "preloadedPages",
            "body": {
                    "method" : "preload",
                    "params" : {
                                "urls" : [URL_FOR_HAND_OVER_NUMBER_ONE, URL_FOR_HAND_OVER_NUMBER_TWO]
                    },
            },
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
                    .GeneralHandler
                    .postMessage(
                        JSON.stringify({ 
                            "jsonrpc" : "2.0",
                            "handler": "preloadedPages",
                            "body": {
                                    "method" : "redirect",
                                    "params" : {"url" : URL_FOR_HAND_OVER_NUMBER_TWO
                                    },
                            },
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
import { useEffect } from "react";

import "../styles/title.css";

const URL_FOR_HAND_OVER_NUMBER_ONE = 'https://dzen.ru/';
const URL_FOR_HAND_OVER_NUMBER_TWO = 'https://dzen.ru/pogoda/saint-petersburg?lat=59.938951&lon=30.315635';

const POST_MESSAGE_DELAY = 5000;

export default function OursComponent(){

    useEffect( () => {
        if (window.webkit?.messageHandlers?) {
            window
                .webkit
                .messageHandlers
                .GeneralHandler
                .postMessage(
                    JSON.stringify({
                        "jsonrpc" : "2.0",
                        "handler": "preloadedPages",
                        "body": {
                                "method" : "preload",
                                "params" : {
                                            "urls" :[URL_FOR_HAND_OVER_NUMBER_ONE, URL_FOR_HAND_OVER_NUMBER_TWO]
                                },
                        },
                        "id" : 3,
                    })
                );

            setTimeout(() => { 
                window
                    .webkit
                    .messageHandlers
                    .GeneralHandler
                    .postMessage(
                        JSON.stringify({ 
                            "jsonrpc" : "2.0",
                            "handler": "preloadedPages",
                            "body": {
                                    "method" : "redirect",
                                    "params" : {"url" : URL_FOR_HAND_OVER_NUMBER_TWO
                                    },
                            },
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
Задача: 
Для нашего сайта, мы хотим реализовать в браузере предзагруженную страницу с Дзен-Видео. 
При этом, нам важно, чтобы  работала передача токена, чтобы пользователя не разлогинило при переходе на страницу с видео. 
Изначально пользователь авторизован и находится на странице Дзен-Статьи.

Решение:

1) Сохраняем две ссылки.
```javascript
const URL_FOR_HAND_OVER_NUMBER_ONE = 'https://dzen.ru/articles';
const URL_FOR_HAND_OVER_NUMBER_TWO = 'https://dzen.ru/video';
```

2) Отправляем браузеру сохранить в хранилище  `loonaStorage` токен.
```javascript
window
    .webkit
    .messageHandlers
    .GeneralHandler
    .postMessage(
        JSON.stringify({
            "jsonrpc": "2.0",
            "handler": "loonaStorage",
            "body": {
                    "method" : "set",
                    "params" :  {
                        "key" : "token",
                        "value" : JSON.stringify({ "access_token": 'string',
                                                    "refresh_token": 'string',
                                                    "scope": 'string',
                                                    "id_token": 'string',
                        })
                    },
            },
            "id" : 1,
        })
    );
```

3) Получаем токен по ключу `token` из хранилища браузера `loonaStorage`.
```javascript
window
    .webkit
    .messageHandlers
    .GeneralHandler
    .postMessage(
        JSON.stringify({
            "jsonrpc" : "2.0",
            "handler": "loonaStorage",
            "body": {
                    "method" : "get",
                    "params" : {"key" : "token"},
            "id" : 2,
        })
    );
```

4) Отправляем браузеру информацию о том, какие ссылки нужно предзагрузить. Ссылки на Дзен-Статьи и Дзен-Видео.
```javascript
window
    .webkit
    .messageHandlers
    .GeneralHandler
    .postMessage(
        JSON.stringify({
            "jsonrpc" : "2.0",
            "handler": "preloadedPages",
            "body": {
                    "method" : "preload",
                    "params" : {"urls" : [URL_FOR_HAND_OVER_NUMBER_ONE, URL_FOR_HAND_OVER_NUMBER_TWO]},
        "id" : 3,
        }
          })
      );
```
5) Отправляем браузеру информацию на какую ссылку надо перейти(`Ссылка на Дзен-Видео`).
```javascript
window
    .webkit
    .messageHandlers
    .GeneralHandler
    .postMessage(
        JSON.stringify({
            "jsonrpc" : "2.0",
            "handler": "preloadedPages",
            "body": {
                    "method" : "redirect",
                    "params" : {"url" : URL_FOR_HAND_OVER_NUMBER_TWO},
            "id" : 4,
            }
        })
    );
```
6) При клике пользователя по <ins>ссылке на Дзен-Видео</ins>, браузер отобразит предзагруженную страницу с погодой. Пользователь останется в авторизованной зоне.  

Д
Данный пример подробнее, с отображением двух ссылок на странице и передачей токена для предзагружаемой страницы Дзен-Видео. 
Инструменты: React.
```javascript

import { useEffect } from "react";

import "../styles/title.css";

const URL_FOR_HAND_OVER_NUMBER_ONE = 'https://dzen.ru/articles';
const URL_FOR_HAND_OVER_NUMBER_TWO = 'https://dzen.ru/video';

export default function OurComponentWithAuthentification(){
  
    const didRecieveLoonaStorageResponse = (event) => {
      /*  Описываем поход с данными(токеном) из loonaStorage на наш сервер для аутентификации и авторизации.
          Ответ(объект) от сервера приходит в формате `JSON string`. Объект в формате JSON-RPC.
      */
    }

    useEffect( () => {
      if (window.webkit?.messageHandlers?) {
            window
                .webkit
                .messageHandlers
                .GeneralHandler
                .postMessage(
                    JSON.stringify({
                        "jsonrpc" : "2.0",
                        "handler": "loonaStorage",
                        "body": {
                                "method" : "set",
                                "params" :  {
                                            "key" : "token",
                                            "value" : JSON.stringify({ "access_token": 'string',
                                                                        "refresh_token": 'string',
                                                                        "scope": 'string',
                                                                        "id_token": 'string',
                                                                    })
                                            },
                                      "id" : 1,
                    })
                );
            window
                .webkit
                .messageHandlers
                .GeneralHandler
                .postMessage(
                    JSON.stringify({
                        "jsonrpc" : "2.0",
                        "handler": "loonaStorage",
                        "body": {
                                "method" : "get",
                                "params" : {"key" : "token"},
                        "id" : 2,
                        }
                    })
                );

            window
                .webkit
                .messageHandlers
                .GeneralHandler
                .postMessage(
                    JSON.stringify({
                        "jsonrpc" : "2.0",
                        "handler": "preloadedPages",
                        "body": {
                                "method" : "preload",
                                "params" : {"urls" : [URL_FOR_HAND_OVER_NUMBER_ONE, URL_FOR_HAND_OVER_NUMBER_TWO]},
                            "id" : 3,
                        }
                    })
                );

            window
                .webkit
                .messageHandlers
                .GeneralHandler
                .postMessage(
                    JSON.stringify({
                        "jsonrpc" : "2.0",
                        "handler": "preloadedPages",
                        "body": {
                                "method" : "redirect",
                                "params" : {"url" : URL_FOR_HAND_OVER_NUMBER_TWO},
                        "id" : 4,
                        }
                    })
                );
            
    }

    window.addEventListener('message', didRecieveLoonaStorageResponse);
    return () => window.removeEventListener('storage', didRecieveLoonaStorageResponse);
    
  },[])

    return (
      <>
          <a
              href='https://dzen.ru/articles'
          >
              Ссылка на Дзен-Статьи
          </a>
          <br/>
          <a 
              href='https://dzen.ru/video'
          >
              Ссылка на Дзен-Видео
          </a>
    </>)
}
```