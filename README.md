

# Разработка RESTful API на Node.js
## Foreword
Итак, цель данного практикума - разработка RESTful API в среде Node.js.
Посмотрим, что же говорят о Node.js на [википедии](https://ru.wikipedia.org/wiki/Node.js):

> **Node** или **Node.js** - программная платформа, основанная на движке **V8** (транслирующем JavaScript в машинный код), превращающая JavaScript из узкоспециализированного языка в язык общего назначения

Дання программная платформа, основанная на движке **Javascript V8**, позволит нам выполнять Javascript код в любой среде.

Как же  **Node** превращает наш старый-добрый, клиентский JS в язык общего назначения?

 - Набор встроенных стандартных библиотек портированных из `C-lang`
 - Пакетный менеджер **NPM** 
 - Кроссплатформенность 

Встроенные библиотеки **Node** дают отправную точку для построения чего угодно - на базе платформы **Node** разрабатываются сервисы от ПО умных домов и дронов до высоко-нагруженных серверов с *10к+* одновременных подключений.

> **NPM** - Встроенный пакентый менеджер, который позволяет вам использовать инструменты сторонних разработчиков, типа [`babel`](https://www.npmjs.com/package/babel), [`lodash`](https://www.npmjs.com/package/lodash) или тот-же [`jquery`](https://www.npmjs.com/package/jquery) при разработке своих приложений или публиковать свои. Стоит помнить, что **сторонние библиотеки и фреймворки не будут работать в браузере если они подключаются с помощью ф-ции `require`**.

> [**REST**](https://habrahabr.ru/post/38730/)(Representational state transfer) - Это стиль архитектуры программного обеспечения для распределенных систем, таких как World Wide Web, который, как правило, используется для построения веб-служб. Термин REST был введен в 2000 году Роем Филдингом, одним из авторов HTTP-протокола. Системы, поддерживающие REST, называются RESTful-системами.

## План практикума

 1. ТЗ
 2. Выбор инструментов
 4. Настройка окружения и рабочей среды
 5. Немного серверного-кода

## ТЗ
Нам дано такое **техническое задание**: нужно разработать RESTful API - Новостной сервис/агрегатор по шаблону проектирования [**MVC**](https://ru.wikipedia.org/wiki/Model-View-Controller) на базе платформы **Node.js** и данных  [**The Guardian**](https://www.theguardian.com/)

## Выбор инструментов
### Сервер
Основой нашего сервиса будет веб-фреймворк - [`epxress.js`](http://expressjs.com/ru/):

> Express
> : это минималистичный и гибкий веб-фреймворк для приложений Node.js, предоставляющий обширный набор функций для мобильных и веб-приложений. Имея в своем распоряжении множество служебных методов HTTP и промежуточных обработчиков, создать надежный API можно быстро и легко. Express предоставляет тонкий слой фундаментальных функций веб-приложений, которые не мешают вам работать с давно знакомыми и любимыми вами функциями Node.js

В нашем же случае, **Express** позволит нам легко и быстро *поднять* и сконфигурировать API-сервис.

Дополнительные инструменты:

 - [`async`](https://www.npmjs.com/package/async) - инструмент который позволяет немного упростить и привести ваш асинхронный код к более красивому и читаемому виду
 - [`http`](https://nodejs.org/api/http.html)/[`https`](https://nodejs.org/api/https.html) - стандартный http-модуль, который поставляется вместе с Node.js

### База данных и модель данных
В роли базы данных будем использовать [**MongoDB**](https://www.mongodb.com):

> **MongoDB**
> : (от англ. *humongous — огромный*) — документоориентированная система управления базами данных (СУБД) с открытым исходным кодом, не требующая описания схемы таблиц. Написана на языке `C++`

Выбор **MongoDB** в роли основной базы данных обоснован высокой производительностью СУБД и легкостью в манипуляции данными, а так-же то, что ТЗ соответствует основному use-case'у для данной СУБД, а именно - хранение и управление данными с неявной структурой.

В качестве менеджера моделей данных будем использовать [`mongoose`](http://mongoosejs.com/) - библиотекa-интерфейсо, позволяющая создавать удобные и функциональные схемы документов.
 
## Настройка окружения и рабочей среды
План ~~побега~~ действий таков:

 1. Создание структуры рабочей директории 
 2. Установка и настройка Node.js, NPM и MongoDB
 3. Инициализация проекта и установка зависимостей
 5. Конфигурирование MongoDB для проекта

### Создание структуры рабочей директории 
```
project-root/
	`-- etc/					-> configs, logs
	`-- node_modules/			-> npm packages
		`-- async/
		`-- express/
		`-- ...
	`-- public/					-> client-side statics
		`-- templates/			-> client-side templates
		`-- vendor/				-> client-side packages
			`-- bootstrap/
			`-- bootswatch/
			`-- ...
	`-- server/					-> server-side logic
		`-- controllers/		-> server-side controllers
		`-- lib/				-> self-written libs
		`-- models/				-> server-side models
		`-- routes/				-> api routes
		`-- views/				-> server-side templates
			`-- layouts/		-> server-side layouts
```
Папка `node_modules/` генерируются автоматически - создавать её вручную **не нужно**, и изменять без должного понимания **не рекомендуется**

### Установка и настройка Node.js, NPM и MongoDB

 - [Установка Node.js(Brad Traversy)](https://www.youtube.com/watch?v=tlntE8fe6u4)
 - [Установка MongoDB по версии METANIT](http://metanit.com/nosql/mongodb/1.2.php)
- [Установка в UNIX-среде](http://stackoverflow.com/questions/28945921/e-unable-to-locate-package-mongodb-org)

Для работы в среде Cloud9:

```bash
$ sudo apt-get install -y mongodb-org
```

### Инициализация проекта и установка зависимостей
Приступим к инициализации проекта. Для того чтобы иметь возможность устанавливать `npm` модули для этого нужно инициализировать соответствующие конфигурации проектов.
Для `npm` это:

```bash
$ npm init
```
или:

```bash
$ npm init -y
```

![enter image description here](https://snag.gy/ByTLia.jpg)

Разница между ними только в том, что `npm init` с флагом `-y` инициализирует проект с указанием полей конфигурационного файла по-умолчанию, а в первом случае вы можете заполнить их по своему усмотрению. Узнать больше о `npm` конфигурации можно [тут](https://docs.npmjs.com/cli/config) и [тут](https://habrahabr.ru/post/243335/).

> **Зачем вообще нужны пакетные менеджеры?**
> Менеджеры пакетов упрощают установку и обновление зависимостей проекта, то есть сторонних библиотек, которые он использует: jQuery, Fotorama, все, что используется на твоем сайте и написано не тобой. Хождение по сайтам библиотек, скачивание и распаковка архивов, копирование файлов в проект — все это заменяется парой команд в терминале.
> У многих языков программирования есть стандартные менеджеры пакетов, которыми разработчики пользуются для установки всех библиотек: gem у Ruby, pip у Python и другие.

Для того чтобы установить какой-нибудь пакет/модуль/плагин/библиотеку/фреймворк/whatever нужна лишь одна команда `npm`:

```bash
$ npm install module_name library_name ...
``` 

`npm` скачает заархивированный модуль со всеми его зависимостями, распакует и установит в автогенерируемую под-директорию вашего проекта `node_modules`.

Команда `npm install` имеет ряд важных флагов, среди которых `-g`, `-S`, `-D`:

Флаг `-g` сообщает  `npm`, что данные модуль/модули нужно установить **глобально**, что означает что они в дальнейшем будут доступны из любого места на вашем ПК. Чаще всего данный флаг используется для установки CLI-[инструментов](https://habrahabr.ru/post/126605/)

Флаги `-S` и `-D` сообщают `npm` о том, что после установки пакетов нужно записать их в список зависимостей `package.json`. Разница между ними в том, что `-S` используется для сохранения пакета как основной зависимости т.е. такой, которая  важна для функционала вашего проекта а `-D` для сохранения пакета как зависимость которая полезна для разработки, но никак не влияет на работоспособность приложения, это могут быть: сборщики, разного рода компиляторы, инструменты тестирования и др.

Флаг `-S` является алиасом для флага `--save`, и `-D` для `--save-dev`, и в конце-концов у самой команды `install` есть сокращенный вариант `i`, т.е. и такая запись будет правильной:

```bash
$ npm i -S some-module
```  

И такая:

```bash
$ npm install --save some-module
```

Теперь соберем в одну кучу весь список зависимостей которые понадобятся для реализации нашего сервиса и установим их:

 - `async`
 - `express`
 - `mongoose`

### Конфигурирование MongoDB для проекта
**Mongo** - эта СУБД из разряда **No-SQL** баз данных - она хранит данные не  в виде классических SQL таблиц, таких как в PostgreSQL, MySQL  или MSSQL, но взамен все данные в MongoDB представляются в виде [`JSON`](https://ru.wikipedia.org/wiki/JSON)-объектов и хранятся в виде [`BSON`](https://ru.wikipedia.org/wiki/BSON) - формата бинарного представления простейших структур данных JavaScript. Так как это не *старые-добрые* таблички в SQL, то и композиция данных тут другая: в то время, как типичная база данных SQL состоит из таблиц -  база данных MongoDB состоит из **коллекций**; вместо строк в таблицах SQL коллекция MongoDB содержит **документы**, а вместо колонок строки в SQL таблице документ принимает вид JS объекта. Полями **документа** по стандарту `JSON` могут быть строки, числа, массивы и объекты.

Для того, чтобы мы могли работать с базой данных, нужно наладить соединение с ней. Если вы правильно установили **MongoDB**, то, для начала, нужно запустить [*демон*](https://en.wikipedia.org/wiki/Daemon_(computing)) [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod.exe/). Находится он, обычно, в директории `${MongoDB src}/Server/${MongoDB version}/bin/`

Для удобства вы можете создать директорию `.data` в корне своего проекта и исполняемый файл `mongod.bat` для Windows или `mongod.sh` с правами доступа `a+x` для `UNIX`-систем с таким содержанием:

```bash
${path_to_mongod} --dbpath=.data
```

Где `${path_to_mongod}` - путь до исполняемого файла `mongod` на вашей системе.

Для запуска в среде Cloud9:

```bash
$ mkdir .data
$ echo 'mongod --bind_ip=$IP --dbpath=.data --nojournal --rest "$@"' > mongod
$ chmod a+x mongod
```

И запустить вызовом нашего простого скрипта:

```bash
$ ./mongod
```

![UNIX](https://snag.gy/lOLYVe.jpg)

В среде Cloud9:

![enter image description here](https://snag.gy/FkiU94.jpg)

![enter image description here](https://snag.gy/BJgNEn.jpg)

Для работы с **MongoDB** напрямую, можно запустить хост `mongo` в директории `${MongoDB src}/Server/${MongoDB version}/bin/`. 

Если хост запущен успешно, вы увидите:

```bash
MongoDB shell version: 3.2.9
connecting to: test
>
```

Командой `show dbs` хост выведет все доступные базы данных

![enter image description here](https://snag.gy/A460nb.jpg)

Для вывода доступных команд можно ввести

```bash
> db.help()
```

![enter image description here](https://snag.gy/2lZAMJ.jpg)

Далее следует создать базу данных для нашего проекта простой командой:

```bash
> use newsdb
```

## План реализации
Базовый принцип работы RESTful API таков: есть набор роутов(маршрутов), по которым проходит пользователь, и в зависимости от конкретного роута реагирует назначенный контроллер модели.

Что можно понимать под "реагированием"? Известно, что есть несколько основных типов `http`-запросов: `GET`, `POST`, `PUT`, `DELETE`. Так, в зависимости от требуемого функционала контроллер реагирует на запрос и возвращает определенный набор данных(ответ)

В нашем случае, контроллеры будут реагировать только на `GET`-запросы т.к. требуется только получать информацию.

Роут - это, определенный маршрут с параметрами и аргументами который строится над `url` сайта или приложения:

```http
https://www.theguardian.com/world/europe-news
```

Разберем данный адрес, и посмотрим, что тут является роутом, а что нет:

```bash
https://			# протокол
theguardian.com 	# домен
/world/europe-news	# роут
``` 

В данном `URL` мы запросим под-категорию новостей `/europe-news` категории `/world` новостного сайта `guardian.com` по защищенному протоколу подключения `https://`

По функционалу, наш сервис будет схож с [The Guardian](https://www.theguardian.com/international). Он будет хранить и отображать данные, полученные из `http://content.guardianapis.com`

Далее определим основные роуты(маршруты) для нашего сервиса:

```bash
/					# index route
/news/				# news section root
/news?id			# get a single article
/news?sections		# get multiple sections
/news/latest		# latest news section
/news:section		# get a section articles 
``` 

В начале разработки серверной части нам нужно подключить зависимости и настроить наш сервер, как отправную точку для дальнейшей разработки и функционирования сервиса.

Затем нужно описать модель данных, контроллер модели и роутер.

## Немного серверного кода
1. Создание конфигурационного файла
2. `app.js` как стартовая точка сервера
3. Получение данных из `content.guardian.api`
4. Модель данных
5. Контроллеры
6. Роуты
7. Последние приготовления и запуск

### Создание конфигурационного файла
Создадим файл `config.js` в директории `etc`. 
В этом файле мы опишем константы, которые будут использоватся в дальнейшем при разработке и работе сервиса:

```js
var config = {
	SERVER: {
		HOST: process.env.IP || 'localhost',
		PORT: process.env.PORT || 3000
	},
	DATABASE: {
		HOST: process.env.IP || 'localhost',
		NAME: 'newsdb' 
	},
	API: {
		KEY: '...',
		SECTIONS: ['world', 'sport', 'football', 'commentisfree', 'culture', 'business', 'lifeandstyle', 'fashion', 'environment', 'technology', 'travel'],
		FIELDS: ['headline', 'trailText', 'byline', 'main', 'body', 'shortUrl', 'thumbnail']
	}
}

module.exports = config;
```

Для обьекта `API`, в частности для его поля `KEY` нужно [зарегестрировать свой ключ доступа](https://bonobo.capi.gutools.co.uk/register/developer).

Так же, именно последняя строка `module.exports = config` позволяет нам подключать данный файл в другие модули нашего приложения

### `app.js` как стартовая точка сервера
Создадим файл `app.js` в директории `server`. Подключим установленные ранее зависимости и наш ранее созданный  конфиг:

```js
var express      = require('express'),
    path         = require('path'),
    handlebars   = require('express-handlebars'),
    mongoose     = require('mongoose'),
    config       = require('../etc/config');
```

Вы можете заметить тут модуль `path` о котором раньше ничего не говорилось. Модуль [`path`](https://nodejs.org/api/path.html) является встроенным модулем Node.js и содержит утилиты для работы с путями.

Далее инициализируем наше приложение вызовом ф-ции `express`:

```js
var app = express();
```

И запустим сервер простым вызовом метода `express` **`listen`**:

```js
app.listen(config.SERVER.PORT, config.SERVER.HOST, () =>
    console.log(`\nMagic happens on port ${config.SERVER.PORT}`));
```

Что здесь происходит: мы говорим нашему `express`-приложению прослушивать определенный нами `PORT` на определенном `HOST` и использовать данный адрес как отправную точку нашего приложения. Например:

```js
// http://${HOST}:${PORT}
http://localhost:3000/
``` 

**Внимание** - дальнейшие изменения нужно будет вносить после инициализации переменной `app` и до запуска сервера `app.listen()`!

Чтобы проверить, что все мы сделали правильно, сервер нужно запустить простой командой:

```bash
$ node server/app
``` 

![enter image description here](https://snag.gy/b1DifB.jpg)

### Получение данных из `content.guardian.api`
Для того чтобы получить данные из **Guardian API** нам нужно сделать `GET`-запрос к **Guardian** RESTful API. Для этого  мы будем использовать стандартную реализацию `http` в **Node.js** и нашь уже подготовленный конфиг. Посмотреть и потрогать **Guardian API** можно [здесь](http://open-platform.theguardian.com/explore/).

Создадим файл `guardian-get.js` в директории `server/lib` и подключим все что нам потребуется, а именно `http` и `https` модули и нашь конфиг:

```js
var http = require('http'),
	https  = require('https'),
	config = require('../../etc/config');
```

Далее определим главную функцию:

```js
var httpGET = (options, done, err) => { }
``` 

Расшифрую аргументы функции:

 - `options` - обьект конфигурации запроса, будет рассмотрен далее
 - `done` - callback-функция, которая будет вызвана при успешном окончании запроса
 - `err` - callback-функция - обработчик ошибок 

Принцип работы функции будет таким:

 - Определить подключение: защищенное / незащищенное
 - Запустить асинхронное выполнение запроса
 - Собирать данные по мере их поступления
 - По окончании запроса вызвать callback-функцию `done`
 - При возникновении ошибки вызвать обработчик

Приступим к реализации:

```js
var httpGET = (options, done, err) => {
	// Определить подключение: защищенное / незащищенное
	var server = options.port === 443 ? https : http;
	// Запустить асинхронное выполнение запроса
	var req = server.request(options, res => {
		// Строка, в которую мы будем собирать входящие данные
		let output = '';
		// Кодировка, в которой мы хотим получить данные
		res.setEncoding('utf8');
		// Собирать данные по мере их поступления
		res.on('data', chunk => output += chunk);
		// По окончании запроса вызвать callback "done" и передать в него полученные данные
		res.on('end', () => done(JSON.parse(output)));
		// Выведем состояние запроса
		console.log(`Request status: ${options.host}:${res.statusCode}`);
	});

	// При возникновении ошибки вызвать обработчик
	req.on('error', err);
	// Закрыть подключение
	req.end();
}
```

Нет, я не забыл описать тот загадочный объект `options`. Этот объект описывает, или другими словами настраивает наш `http`-запрос. Он должен содержать такие поля: `host`, `port`, `path`, `method`, `headers`

- `host`: базовый адрес, к которому мы обращаемся или URI определяющий путь к запрашиваемому документу
- `port`: порт подключения
- `path`: здесь мы описываем запрос к `host`'у
- `method`:  [вид запроса](https://ru.wikipedia.org/wiki/HTTP#.D0.9C.D0.B5.D1.82.D0.BE.D0.B4.D1.8B) - `GET`, `POST`, `PUT`, `DELETE`
- `headers`:  строки HTTP-сообщения, содержащие разделённую двоеточием пару параметр-значение

Описывать объект `options` мы будем при вызове нашей функции `httpGET` из анонимной функции-обёртки:

```js
module.exports = (query, done, err) => {
	// Шаблон запроса к The Guardian
    var queryTemplate = `/search?show-fields=${config.API.FIELDS.join(',')}&api-key=${config.API.KEY}&${query}`;
    var requestOptions = {
        host: 'content.guardianapis.com',	// API-endpoint
        port: 443,
        path: queryTemplate,
        method: 'GET',
        headers: {
            'Content-Type': 'application/json'
        }
    };

    httpGET(requestOptions, done, err);
};
```

Протестировать этот скрипт можно с помощью [Node REPL](https://nodejs.org/api/repl.html):

```bash
$ node
> 'use strict'
> var get = require('./server/lib/guardian-get');
> get('', console.log.bind(console), console.log.bind(console));
```

Если вы все правильно ввели, то статус запроса должен быть таким:

```bash
> Request status: content.guardianapis.com:200
```

И через некоторое время вы получите ответ от  `content.guardianapis.com`:

```json
{
    "response":{
        "currentPage":1,
        "pageSize":10,
        "pages":189508,
        "total":1895080,
        "userTier":"developer",
        "startIndex":1,
        "results":[
            [ "Object" ],
            [ "Object" ],
            [ "Object" ],
            [ "Object" ],
            [ "Object" ],
            [ "Object" ],
            [ "Object" ],
            [ "Object" ],
            [ "Object" ],
            [ "Object" ]
        ],
        "status":"ok",
        "orderBy":"newest"
    }
}
```

![enter image description here](https://snag.gy/iGsNVR.jpg)

### Модель данных
Хотите пример структуры данных при использовании **Mongoose** с моими кривыми стрелочками?

![enter image description here](http://i66.tinypic.com/4qo35x.jpg)

**Моделью** в нашем проекте будет считаться некоторая сущность-информационная единица, которая будет хранится в базе данных, а именно **MongoDB**, и которая будет иметь поведение, или другими словами содержать в себе несколько вспомогательных методов для работы с БД. Определять модель данных мы будем с помощью библиотеки `mongoose`. **Mongoose** будет оборачивать наши данные в специальную [схему данных](http://mongoosejs.com/docs/guide.html) для дальнейшего их использования нашим сервисом. 

Теперь, когда *демон* успешно **запущен**, нам нужно добавить всего одну строчку в `app.js`:

```js
mongoose.connect(
	`mongodb://${config.DATABASE.HOST}/${config.DATABASE.NAME}`, 
	err => {
		if (err) throw err;
		console.log("Magic connected to database");
	});
```

Первым аргумент - `url` подключения к БД, второй - `callback` который уведомит нас об ошибке или успешном выполении

![enter image description here](https://snag.gy/m9tnzS.jpg)

#### Схема данных

Пришло время занятся схемой нашей будущей модели. Схема или `Mongoose.Schema` позволяет нам функционально и семантически разметить тип хранимых нами в БД данных. Процесс описания схемы чем-то схож с созданием новой таблицы в SQL-подобных СУбД - нам нужно описать название полей объекта, их тип и другие параметры. Но это только аналогия, документы **MongoDB** в лице **Mongoose**-схем штука порядком мощнее, но это вы и сами в дальнейшем увидите. А сейчас приступим к схеме. Создадим файл `guardian-content.model.js` в директории `server/models` с таким содержимым:

```js
"use strict";

var mongoose = require('mongoose');

var GuardianContentSchema = new mongoose.Schema({
	id: {
		type: String,
		index: true
	},
	sectionId: String,
	type: String,
	webPublicationDate: Date,
	title: String,
	webUrl: String,
	fields: {
		headline: String,
		trailText: String,
		byline: String,
		main: String,
		body: String
	}
});
```

Внимание - тип поля `fields` - объект, а его значением является вложенная схема. Да - так тоже можно, это же `JSON`. Это же `JS` в конце-концов - у нас тут все можно ;)

Мы можем расширить определение полей немногим количеством опциональных параметров, например: 

```js
var GuardianContentSchema = new mongoose.Schema({
	id: {
		type: String,
		index: true
	},
	sectionId: {
		type: String,
		required: true
	},
	type: {
		type: String,
		default: 'article'
	},
	webPublicationDate: {
		type: Date,
		default: Date.now
	},
	title: {
		type: String,
		required: true,
		trim: true
	},
	webUrl: {
		type: String,
		required: true
	}
}
```

Здесь:

 - `index` указывает, что по этому полю документы будут индексироваться
 - `required` задает поле как обязательное для заполнения
 - `default` - значение по умолчанию
 - `trim` - имплементация все того-же [`String.prototype.trim()`](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/String/Trim) 

Но это был демонстрационный пример, продолжать мы будем с первой, лаконичной версией.

С описанием схемы/модели/прототипа данных мы закончим, передав в конструктор `Mongoose.Schema()` дополнительный параметр опций. Добавим данный сниппет кода перед определением схемы:

```js
var GuardianContentSchemaOptions = {
	timestamps: {},
	safe: true
}
```

Данные опции позволяют присваивать даты создания и изменения конкретной модели. Передадим их в конструктор:

```js
var GuardianContentSchema = new mongoose.Schema({
	// Schema...
}, GuardianContentSchemaOptions);
``` 

Вот теперь можно сказать, что со схемой данных мы закончили. Теперь нужно определить модель данных:

```js
var GuardianContentModel = mongoose.model("news", GuardianContentSchema);

module.exports = GuardianContentModel;
```

Здесь мы определяем `mongoose`-модель с помощью конструктора `mongoose.model`, который принимает два параметра - название модели и схему данных. Далее, при создании первого объекта модели в нашей базе данных автоматически создастся коллекция с с именем модели во множественном числе, т.е. в нашем случае это будет так и останеться - "news". Но если бы мы определяли модель под названием "apple" или "bill" то создались бы коллекции под названием "apples" и "bills" соответственно.  

Теперь можем проверить работоспособность нашей модели

```bash
$ node
> var mongoose = require('mongoose');
> var Model = require('./server/models/guardian-content-model.js');
> new Model({id: "sport/article", sectionId: "sport", type: "article", webPublicationDate: Date.now(), title: "New section", webUrl: "http://google.com", fields: {}});
```

![enter image description here](https://snag.gy/vz86xl.jpg)

#### Поведение модели и вспомогательные методы
**Mongoose** определяет несколько типов  функций для манипуляций данными:

 - Встроенные методы
 - Пользовательские методы
 - Статические методы
 - Вспомогательные запросы

Мы будем использовать несколько встроенных методов библиотеки **Mongoose**:

 - `find()` и `findOne()`
 - `sort()`
 - `limit()`
 - `save()`

Для ф-ций `find()` и `findOne()` аргументом является объект-конфиг поиска. Пример для нашей модели данных:

```js
// Получить статьи только этого года
var query = { 
	type: "article",
	webPublicationDate: new Date().getFullYear() 
} 

Model.find(query)
	.exec(console.log.bind(console));
```

Мы будем определять свои вспомогательные функции запросов. Об остальных типах вы можете узнать из [документации](http://mongoosejs.com/docs/guide.html)

Нам понадобятся четыре ф-ции:

 - `byContentId`
 - `latest`
 - `bySections`
 - `bySection`

Синтаксис использования вспомогательных ф-ций будет примерно таков:

```js
Model.find()
	.byContentId("i'm an id of section")
	.exec(callback);
```

В ф-цию `exec` передается `callback` с двумя аргументами - объект ошибки и объект найденных данных:

```js
let searchCallback = (err, data) => {
	if (err) {
		throw err;
	} else {
		// Do something with data
	}
}
```

Итак, приступим к реализации. Для начала изобразим ф-цию `byContentId`:

```js
GuardianContentSchema.query.byContentId = function(id) {
	return this.findOne({ 
		id: id
	});
};
```

Здесь метод `Model.findOne` является встроенным, и из названия можно догадаться, что возвращает он только один экземпляр документа с переданными в него опциями.  

```js
GuardianContentSchema.query.latest = function() {
	return this.find()
		.sort('-date')
		.limit(10);
};
```

Метод вернет последние документы по дате отсортированные по убыванию

```js
GuardianContentSchema.query.bySections = function(sections) {
	return this.find({ 
			sectionId: { 
				$in: sections.split(',') 
			}
		})
		.limit(20);
};
```

Данный метод вернет документы по секциям, определенным в массиве `sections`. 
И наконец, получение статей по определенной секции:

```js
GuardianContentSchema.query.bySection = function(section) {
	return this.find({ 
			sectionId: section 
		})
		.limit(10);
};
```

**Внимание**, определить данные методы нужно **перед** определением модели.

После определения модели, добавим ей еще один метод - `spread`, который упростит нашу жизнь в будущем:

```js
GuardianContentModel.spread = (items, done) =>
	items.forEach(item =>
		new GuardianContentModel(item)
			.save(done));
```

Он сохраняет множество/массив элементов и принимает два аргумента:
- `items` - массив `JS` объектов, который нужно сохранить
- `done` - ф-ция которая выполнится после сохранении **каждого** документа  

И того, с схемой и моделью данных мы закончили:

```js
// server/models/guardian-content-model.js
"use strict";

var mongoose = require('mongoose');

var GuardianContentSchemaOptions = {
	timestamps: {},
	safe: true
}

var GuardianContentSchema = new mongoose.Schema({
	id: {
		type: String,
		index: true
	},
	sectionId: String,
	type: String,
	webPublicationDate: Date,
	title: String,
	webUrl: String,
	fields: {
		headline: String,
		trailText: String,
		byline: String,
		main: String,
		body: String
	}
}, GuardianContentSchemaOptions);

GuardianContentSchema.query.byContentId = function(id) {
	return this.findOne({ 
		id: id 
	});
};

GuardianContentSchema.query.latest = function() {
	return this.find()
		.sort('-date')
		.limit(10);
};

GuardianContentSchema.query.ofSections = function(sections) {
	return this.find({ 
			sectionId: { 
				$in: sections.split(',') 
			}
		})
		.limit(20);
};

GuardianContentSchema.query.ofSection = function(section) {
	return this.find({ 
			sectionId: section 
		})
		.limit(10);
};

var GuardianContentModel = mongoose.model("news", GuardianContentSchema);

GuardianContentModel.spread = (items, done) =>
	items.forEach(item =>
		new GuardianContentModel(item)
			.save(done));

module.exports = GuardianContentModel;
```

### Контроллеры

#### Index controller

Index-контроллер будет отвечать за переход к корню сайта, т.е. к роуту `/`. По функционалу, `index`-контроллер обычно следит за состоянием авторизации пользователя, который проходит по роута, кеширование или инициализацию БД.

В нашем случае, `index`-контроллер будет иметь только один метод `reserve`, который будет обновлять БД актуальным контентом из **Guardian API**.

Создадим файл `index.controller.js` в директории `server/controllers` и первым делом подключим зависимости. Нам понадобятся модуль `async`, наш конфигурационный файл, `http-get`-функция и наша модель данных:

```js
"use strict";

var async       = require('async');
var config      = require('../../etc/config');
var guardianGet = require('../lib/guardian-get');
var Model       = require('../models/guardian-content.model');
```

Т.к. метод в данном контроллере будет только один, то мы можем определить его сразу в объект `module.exports`:

```js
module.exports.reserve = () => {
	// Whatever...
};
```

Итак, что должен делать наш метод? Нам нужно получить данные(статьи, посты и т.д.) по каждому новостному-разделу сайта `The Guardian` и сохранить их в БД. Но как нам это сделать?

Самое очевидное, что первое приходит на ум - это пройтись по массиву названий секций, получить каждую по очереди и сохранить в БД. Так и сделаем:

```js
module.exports.reserve = () =>
	config.API.SECTIONS.forEach(section =>
		guardianGet(`section=${ section }`, (data, err) => {
			if (err) throw err;
			Model.spread(data.response.results, (err, doc) => 
				console.log(err ? 
					err : `[${doc.id}] saved to ${config.DB.NAME}`))
		}));
```

Ну вот, неплохой-такой спагетти-код. Но тут ничего сложного пока нет. Мы проходим итератором по массиву разделов и для каждого формируем `GET`-запрос передавая первым аргументом в ф-цию `guardianGet` строку запроса. По окончании запроса, т.е. получении данных по каждому разделу сохраняем множество статей в БД с помощью ф-ции `spread`. Но этот весь этот код, включая ф-цию `Model.spread`, будет работать **синхронно**, т.е. пока, на данный момент при срабатывании данного контроллера будем **синхронно** получать 110 статей за раз. 

Допустим, нашим приложением будет пользоваться только один пользователь и обрабатывать 100-300 статей в сутки не такая и большая нагрузка, но если таких пользователей будет 300 или 3000? Посчитать разницу не сложно, а сервер нашего приложения  будет только **один**. 

Т.е. ситуация: пользователь `А` заходит на наш сайт, контроллер обновляет базу данных **синхронно**. Примерно в тот-же момент наш сайт посещает еще один пользователь `B` и на его сессию контроллер срабатывает так-же. Только один момент - контроллер ждет, пока обновление БД закончится после посещения сайта пользователем `A`, и только потом начнет получать и обрабатывать данные для сессии пользователя `B`. А если вернутся в начало, и если, примерно в тот-же момент, что и пользователь `A` на наш сайт заходят еще 50? Все - приложение для клиентов повиснет, и никто из оставшихся 49 ждущих пользователей не будет ждать загрузки контента, а может и пользователь `А` не будет. 

В таком случае нам нужно как-то выходить из ситуации, а именно с помощью **асинхронных** операций. Мы можем изменить архитектуру нашего сервера таким образом, что определенное множество операций не будут блокировать друг-друга а выполняться *независимо*.

Начнем с метода `spread` нашей модели. На данный момент он выглядит так:

```js
GuardianContentModel.spread = (items, done) =>
	items.forEach(item =>
		new GuardianContentModel(item)
			.save(done));
```

Синхронность операций в этом методе выражается прямо в его алгоритме:

 1. Береться `n` елемент
 2. На его основе инициализируется модель
 3. Модель сохраняется в базу данных

Это привычный нам поток выполнения инструкции любой стандартной программы, но проблемы начинаются с первых строк: в то время, как проводятся манипуляции над `n` элементом массива, `n+1` элемент ожидает окончания выполнения операций над элементом `n`. В данном случае это не кажется столь критичным, так как и **JS**, и **MongoDB** довольно быстры. Ну, хорошо когда с производительностью ваших инструментов проблем нет, но данная проблема содержится не в инструментах, а в потоке данных, точнее в их количестве.

> Чем больше данных, тем больше время их обработки!

Мы можем немного расширить реализацию уже существующего алгоритма с помощью библиотеки `async`. Не забудьте **подключить** данную библиотеку!

```js
GuardianContentModel.spread = (items, done) => {
	var tasks = items.map(item => function task() {
		return new GuardianContentModel(item).save(done);
	});

	async.parallel(tasks, 
		() => console.log('Data saved successfuly!'));
};
``` 

Здесь мы формируем массив задач на каждую операцию инициализации модели, т.е. в конце-концов мы получим массив синхронных функций которые нужно будет запустить асинхронно. Во время  Запускать мы их будем с помощью метода `parallel`. Вот и все, теперь данный метод будет запускаться асинхронно. 

Так же сделаем рефакторинг нашего `index`-контроллера:

```js
module.exports.reserve = () => {
	var saveToDatabase = (data, err) =>
		Model.spread(data.response.results, (err, doc) => 
			console.log(err ? err : `[${doc.id}] saved to ${config.DB.NAME}`));

	var asyncTasks = config.API.SECTIONS.map(section => 
		function task() {
			return guardianGet(`section=${ section }`, saveToDatabase);
		});
	
	async.parallel(asyncTasks, () => 
		console.log('Data reserved successfuly'));
};
```

#### News controller

Этот контроллер будет контроллером модели. Он будет той самой прослойкой между бизнес-логикой и данными нашего приложения.

В данном случае, нам нужны методы, которые будут в определенном случае обращаться к модели для получения данных:

 - `get` - будет отвечать за обработку определенной статьи или множества разделов
 - `getSection` - будет отвечать за получение статей из определенного раздела 
 - `getLatest` - будет отвечать за получение свежих новостей

Каждый из методов будет функцией вида:

```js
var anotherMethod = (req, res) => {
	// Whatever
}
```

Такие ф-ции являются неким *шаблоном* для работы с роутами `express`-фреймворка. В дальнейшем, когда мы будем связывать наши роутеры с нашими контроллерами, в аргументы методов будут передаваться специальные объекты [`Request`](http://expressjs.com/ru/4x/api.html#req) и [`Response`](http://expressjs.com/ru/4x/api.html#res).

Объект `Request` представляет `http`-запрос и имеет свойства присущие ему - строку запроса, тело запроса, заголовки и [т.д.](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html)

Объект `Response` - инструмент отправки данных в `express`. Основные [методы](http://expressjs.com/ru/4x/api.html#res) которые мы будем использовать:  

 - `status` - отправляет клиенту статус его запроса
 - `json` - передает данные в формате `JSON`

Создадим файл `index.controller.js` в директории `server/controllers` и подключим нашу модель.

Можно сразу вынести функцию-обработчик полученных данных из БД:

```js
var handleData = (err, data, res) => {
	if (err) {
		res.status(404);	// not found
		console.error(err);
	} else {
		res.status(200)
			.json(data);	// ok/found
	}
};
``` 

Далее определим три вышеописанных метода данного контроллера:

```js
var get = (req, res) => {
	if (req.query.id) {
		Model.find()
			.byContentId(req.query.id)
			.exec((err, data) =>
				handleData(err, data, res));
	} else {
		Model.find()
			.bySections(req.query.sections)
			.exec((err, data) =>
				handleData(err, data, res));
	}
};

var getSection = (req, res) =>
	Model.find()
		.bySection(req.params.section)
		.exec((err, data) => 
			handleData(err, data, res));

var getLatest = (req, res) =>
	Model.find()
		.latest()
		.exec((err, data) => 
			handleData(err, data, res));
``` 

И осталось только экспортировать этот контроллер:

```js
module.exports = {
	get: get,
	getSection: getSection,
	getLatest: getLatest
}
```

### Роуты

Определение роута в `express` выглядит таким образом:

```js
Express.Router.method(path, handler);
```

Набор `handler`-ов мы определили ранее в виде контроллеров. В таком случае осталось привязать их у определенным маршрутам.

Создадим два файла в директории `server/routes/`: `index.route.js` и `news.route.js`

Для `index.route.js` введем: 

```js
"use strict";

var router     = require('express').Router();
var controller = require('../controllers/index.controller');

router.get('/', (req, res) => {
	res.status(201)		// 201 - Webpage created
		.render('home');
	controller.reserve();
});

module.exports = router;
```

Здесь мы создаем экземпляр `express`-роутера, настраиваем с помощью него и `index`-контроллера маршрутизацию и экспортируем это все как модуль.

Для `news.route.js` поступаем аналогично:

```js
"use strict";

var router     = require('express').Router();
var controller = require('../controllers/news.controller');

router.get('/', controller.get);
router.get('/latest', controller.getLatest);
router.get('/:section', controller.getSection);

module.exports = router;
```

### Последние приготовления и запуск

Теперь осталось подключить наши роутеры в `app.js`. Подключать мы будем их перед вызовом метода `app.listen()`:

```js
var routes = {
	index: require('./routes/index.route'),
	news:  require('./routes/news.route')
}

app.use('/', routes.index);
app.use('/news/', routes.news);
``` 

Теперь, откроем браузер и если введем в адресную строку временный адрес нашего приложения:

```http
http://localhost:3000/
``` 

То увидим, что ничего не происходит:

![enter image description here](https://snag.gy/cREdte.jpg)

Но нет, на самом деле происходит:

![enter image description here](https://snag.gy/UuRxJE.jpg)

Просто наш `index`-контроллер ничего не отправляет, а всего лишь выполняет одну, заложенную в него нами функцию - а именно получает данный извне и сохраняет их в БД. Вот и все.

Но если мы перейдем по адресу:

```http
http://localhost:3000/news/latest
```

То получим примерно такую `JSON`-ину:

![enter image description here](https://snag.gy/SEoFwt.jpg)

Если [отформатировать](https://jsonformatter.curiousconcept.com/) её, то увидим, что, насчет модели, все сходится:

```json
[
    {
        "_id": "57fab3ef5fcb400a98c525ea",
        "updatedAt": "2016-10-09T21:17:35.946Z",
        "createdAt": "2016-10-09T21:17:35.946Z",
        "webUrl": "https://www.theguardian.com/fashion/2016/oct/09/the-new-scent-of-a-woman-top-autumn-perfumes",
        "webPublicationDate": "2016-10-09T05:00:02.000Z",
        "id": "fashion/2016/oct/09/the-new-scent-of-a-woman-top-autumn-perfumes",
        "sectionId": "fashion",
        "type": "article",
        "__v": 0,
        "fields":{
            "trailText": "Autumn has never smelled better, with wonderful new scents from established names and some lively newcomers, too, says Eva Wiseman",
            "headline": "The new scent of a woman",
            "body": "<!-- A lot of HTML -->",
            "main": "<!-- Some HTML -->",
            "byline": "Eva Wiseman"
        }
    },
    // Another items next...
]
```

Если попробуем получить по `id`:

![enter image description here](https://snag.gy/zk4esD.jpg)

Несколько разделов

![enter image description here](https://snag.gy/4TZJNp.jpg)

