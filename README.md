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

Файл app.js в корневой директории проекта — это точка входа для всего приложения. Первые несколько строк инициализируют все необходимые модули. Express настроен для обеспечения доступа к статическим файлам, а модуль Socket.IO настроен так, чтобы отслеживать подключения к тому же порту, что и Express. Следующие строки — основа приложения, необходимая для работы сервера.

```node
// Создаем приложение с помощью Express
var app = express();

// Создаем HTTP-сервер с помощью модуля HTTP, входящего в Node.js. 
// Связываем его с Express и отслеживаем подключения к порту 8080. 
var server = require('http').createServer(app).listen(8080);

// Инициализируем Socket.IO так, чтобы им обрабатывались подключения 
// к серверу Express/HTTP
var io = require('socket.io').listen(server);
```
