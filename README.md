Fesenko
=======

# Основные технологии

* **Node.js** - используется для создания серверной части приложения и позволяет использовать фреймворк Express и библиотеку Socket.IO. Кроме того, основная логика реализуется на стороне сервера и оформлена в виде отдельного Node-модуля, о котором мы поговорим позже.

* **Express** - сам по себе Node.js удовлетворяет многим требованиям, предъявляемым при разработке современных веб-приложений. Однако, добавление библиотеки Express позволит нам оптимизировать отдачу статичных файлов (HTML, CSS и JavaScript) пользователям. Мы также используем Express для логирования и реализации окружения, совместимого с Socket.IO.
* **EJS** - простейший HTML шаблонизатор.

* **Socket.IO** - эта библиотека позволяет очень просто реализовать обмен данными между браузером и сервером (в данном случае представляющим собой связку Node.js и Express) в реальном времени. Socket.IO использует протокол веб-сокетов, если он поддерживается браузером по умолчанию. Старые браузеры, такие как IE9, не поддерживают этот протокол — в этом случае Socket.IO будет использовать сокеты через Flash или AJAX-технологию long-polling.

* **MongoDB** - в качестве основного хранилища данных о пользователях и сообщениях.

Клиентский JavaScript – для простоты, на фронтенде используется всего одна JS-библиотека:
* **jQuery** – в основном используется для обработки событий и манипуляций с DOM.

* **CSS** – ...

# Разработка серверной части


Модули Express и Socket.IO не входят в Node.js. Они являются внешними зависимостями, которые должны быть загружены и установлены отдельно. Менеджер пакетов Node.js (npm) сделает это за нас при запуске команды npm install. То же самое он сделает со всеми зависимостями, перечисленными в файле package.json. Файл package.json вы можете найти в корневой директории проекта. Он содержит определяющий зависимости проекта JSON-объект:

```json
"dependencies": {
    "express": "3.x",
    "socket.io":"0.9"
}
```

Файл `app.js` в корневой директории проекта — это точка входа для всего приложения. Первые несколько строк инициализируют все необходимые модули. Express настроен для обеспечения доступа к статическим файлам, а модуль Socket.IO настроен так, чтобы отслеживать подключения к тому же порту, что и Express. Следующие строки — основа приложения, необходимая для работы сервера.

```javascript
// Создаем приложение с помощью Express
var app = express();

// Создаем HTTP-сервер с помощью модуля HTTP, входящего в Node.js. 
// Связываем его с Express и отслеживаем подключения к порту 8080. 
var server = http.createServer(app).listen(app.get('port'), function(){
  console.log('Express server listening on port ' + app.get('port'));
});

// Инициализируем Socket.IO так, чтобы им обрабатывались подключения 
// к серверу Express/HTTP
var io = require('socket.io').listen(server);
```

Весь код серверной части приложения вынесен в отдельные файлы-модули `./routes/*.js`. Этоти файлы подключается в приложение в качестве модуля Node с помощью следующего фрагмента кода (прим.): `var im = require('./routes/im');`. Когда клиент подключается к приложению с помощью Socket.IO, модуль `im` должен выполнять функцию `setConnection` в которую будет передан сокет-объект. За это отвечает следующий фрагмент кода:

```javascript
io.sockets.on('connection', function (socket) {
    im.IM.parentSocket = io.sockets;
    im.IM.setConnection(socket);
});
```

Функция `setConnection` собирает первичную информацию о пользователе, который зашел на сайт:
```javascript
    setConnection: function (socket) {
        this.socket = socket;
        this.socket.handshake.getSession(function(err, session) {
            if (IM.connectedUsers[session._sessionid] === undefined) {
                IM.connectedUsers[session._sessionid] = [];
            }

            IM.connectedUsers[session._sessionid].push(IM.socket.id);
        });
    },
```

Эта информация нужна будет для того, что бы отследить владельца socket-коннекта и дать возможность приложению четко определять конечного клиента для отправки сокета.

# Разработка клиентской части


Когда браузер подключается к приложению, сервер отдает ему файл `index.ejs` из директории `views`. 

Для подключения браузера к серверу Socket.IO требуется клиентская библиотека Socket.io. Так как мы используем Socket.IO через Express, мы можем использовать получить файл библиотеки с сервера с помощью следующего тега:

`<script src="/socket.io/socket.io.js"></script>`

Большая часть логики приложения и весь клиентский код расположен в файле `./public/javascript/im.js`.
Код заключен в Javascript-объект `IM`. Это означает, что все переменные и функции, используемые в приложении, являются свойствами объектов `IM`. Структура клиентского кода выглядит примерно так:

```javascript
IM = {
    socket: null,
    connect: function() {
        // ссылка на Socket.io объект
        this.socket = Node.socket;
        
        // вызов метода-слушателя, который real-time принимает информацию от node.js-сервера
        this.listenEvents();

    },
    listenEvents: function() {
        this.socket.on('events', function(data) {
            if (data.fn) {
                IM[data.fn](data.message);
            }
        });
    },
    send: function(el, prev) {
    
    },
    addMessage: function (data) {
    
    },
    refreshOnline: function(data) {
    
    }
};
```

При первом запуске приложения после окончания загрузки документа вызываются 2 функции: `Node.init('http://localhost'); и IM.connect();`. Первая настраивает подключение через Socket.IO, вторая показывает стартовый экран диалогов в браузере.

Вызов первой функции `Node.init('http://localhost');` инициализирует подключение через Socket.IO между браузером и сервером:

```javascript
Node = {
    socket: null,
    urlIO: null,
    connect: function() {
        try {
            this.socket = io.connect(this.urlIO + ':3000', {
                'connect timeout': 500,
                'reconnect': true,
                'reconnection delay': 500,
                'reopen delay': 500,
                'max reconnection attempts': 10
            });
        } catch (e) {
            throw new RangeError('Unable connect to server. Please, start node.js app!');
        }
    },
    init: function(host) {
        this.urlIO = host;
        this.connect();
    }
};
```

Вслед за этим вызывается функция `IM.connect()`, которая добавляет слушатель событий Socket.IO на клиенте. Этот слушатель работает аналогично слушателю на стороне сервера, но в обратном направлении. Обратите внимание на следующий пример:

```javascript
        this.socket.on('events', function(data) {
            if (data.fn) {
                IM[data.fn](data.message);
            }
        });
```

# Other info


**Обработчик событий (server-side):**
```javascript
    socket.on('events', function(post) {
        socket.handshake.getSession(function(err, session) {
            post['data']['session'] = session;
            im.IM[post.fn](post.data);
        });
    });
```
Данный участок кода "слушает события" и когда с клиента приходит информация, вызывает функцию объекта `im.IM`, название которой хранится в data.fn.

**Структура сообщений client-server/server-client:**
```json
{
    fn: <string>,
    data: <object>
}
```

fn - имя вызываемой функции
data - информация которая передается

**Disconnect socket:**
Внутреннее socket.io-событие `disconnect` вызывается в момент "обрыва" связи между клиентом и сервером. 
В этом случае нам нужно очистить информацию о потзователе из двух хранилишь:
im.IM.connectedUsers - общий объект в котром хранится хэш-таблица о подключенных пользотвателях, структура:
```json
{
    <session_id>: [
        <socket_id1>,
        <socket_id2>,
        <socket_id3>,
    ]
}
```
im.IM.inDialog - объект хранящий в себе информацию о диалогах в которых в данный момент находится пользователь, структура:
```json
{
    <session_id>: [
        <dialog_id1>,
        <dialog_id2>,
        <dialog_id3>,
    ]
}
```

```javascript
    socket.on('disconnect', function () {
        socket.handshake.getSession(function(err, session) {
            if (im.IM.connectedUsers[session._sessionid] !== undefined) {
                im.IM.connectedUsers[session._sessionid].forEach(function(sock, index) {
                    if (sock == socket.id) {
                        im.IM.connectedUsers[session._sessionid].splice(index, 1);
                    }
                });
            }

            if (im.IM.inDialog[session._sessionid] !== undefined) {
                delete im.IM.inDialog[session._sessionid];
            }
        });
    });
```
