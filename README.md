# :fox_face:
## :pushpin: ИНИЦИАЛИЗАЦИЯ ПРОЕКТА - GIT


- [X] `git init` - Инициализация git в данной папке
- [X] `npx gitignore node` - Файл .gitignore
- [X] - добавляем в .gitignore записи: 
`public/app.js
public/vendor.js
sessions/`
- [X] `npm init -y` - Создать файл package.json
`npm install` - Если файл уже есть
- [X] `npx eslint --init` -Установка eslint

`
rules: {
    'react/prop-types': 0,
    'no-console': 0,
    'react/jsx-props-no-spreading': 0,
  },
`



## Библиотеки и пакеты :point_down:
##### УСТАНОВКА ПАКЕТОВ - DEVDEP
Для установки необходимых пакетов в devDependencies пишем: 
`npm i -D` *названия*

- [X] `npm i -D @babel/node @babel/plugin-proposal-class-properties @babel/preset-react @babel/preset-env babel-loader morgan webpack webpack-cli`

##### УСТАНОВКА ПАКЕТОВ - DEP
Для установки необходимых пакетов в dependencies пишем:
`npm i` *названия*
- [ ] `npm i express react react-dom react-router-dom sequelize sequelize-cli pg pg-hstore`


## :pushpin: BABEL
> Babel позволяет решить проблему совместимости Javascript разных версий. Например, с помощью babel можно использовать как require/module.exports, так и import/export.
- [X] Создаём файл `.babelrc` 
- [X] Вставляем конфигурацию:
```Javascript
{
    "presets": [
      ["@babel/preset-env", { "targets": { "node": "current" } }],
      "@babel/preset-react"
    ],
    "plugins": ["@babel/plugin-proposal-class-properties"]
}
```
## :pushpin: WEBPACK
>Webpack нужен для эффективной сборки проектов, всех зависимостей и их компрессии для экономии трафика и ресурсов сервера. Данный конфигурационный файл отвечает за то, что webpack соберёт весь наш проект и все зависимости в 2 файла в папке public: app.js и vendor.js


- [X] Добавляем скрипты в `package.json`
```Javascript
"scripts": {
    "webpack": "webpack -wd eval-source-map",
    "dev": "babel-node src/server.js"
  },
```
- [X] В корне проекта создаём файл `webpack.config.js`
- [X] Вставляем конфигурацию:
```Javascript
const path = require('path');

const config = {
  entry: {
    app: ['./src/components/index.jsx'],
  },
  output: {
    path: path.resolve(__dirname, 'public'),
    globalObject: 'this',
    filename: '[name].js',
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env', '@babel/preset-react'],
          },
        },
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: ['.js', '.jsx', '.json', '.wasm', '.mjs', '*'],
  },
  mode: 'development',
  optimization: {
    splitChunks: {
      cacheGroups: {
        default: false,
        vendors: false,

        vendor: {
          chunks: 'all', // both : consider sync + async chunks for evaluation
          name: 'vendor', // имя чанк-файла
          test: /node_modules/, // test regular expression
        },
      },
    },
  },
};

module.exports = config;
```

## :pushpin: SRC / SERVER
>Все данные проекта будут находиться в папке src. Файл server.js будет являться стартовой точкой запуска express.
Скрипты в package.json добавляем, чтобы можно было разделить запуск webpack и express

- [X] В корне проекта создаём папку src
- [X] Добавляем туда пустой файл `server.js`
- [X] Создаём папки `components` `middlewares` `routes` `utils`
- [X] Добавляем в `package.json` скрипты запуска сервера и сборки webpack:
```Javascript 
 "scripts": {
    "webpack": "webpack -wd eval-source-map",
    "dev": "babel-node src/server.js",
  },

```
- [X] В папке src/routes -> разбиваем логику роутов на разные файлы
-
##### :paperclip: Пример server.js
```Javascript 
import express from 'express';// Подключение библиотеки express
import morgan from 'morgan';
import path from 'path';// Подключаем переменную path для того что бы путь был найден при запуске из другого места
import session from 'express-session';
import store from 'session-file-store';
import jsxRender from './utils/jsxRender';// Импорт кастомного рендера jsx-файлов
import authRouter from './routes/authRouter';
import { User } from '../db/models';
import isAuth from './middlewares/isAuth';

const app = express();
const PORT = 3000;

app.engine('jsx', jsxRender);// Регистрируем движок для рендера
app.set('view engine', 'jsx');
app.set('views', path.join(__dirname, 'components'));// Указываем путь до компонентов

app.use(express.urlencoded({ extended: true }));
app.use(express.static('public')); // Подключение middleware, который отдаёт клиенту файлы из папки или (app.use(express.static(path.join(__dirname, 'public')));
)
app.use(express.json());// Подключение middleware, который парсит JSON от клиента
app.use(morgan('dev'));

const FileStore = store(session);

const sessionConfig = {
  name: 'user_sid', // Имя куки для хранения id сессии. По умолчанию - connect.sid
  store: new FileStore(),
  secret: 'oh klahoma', // Секретное слово для шифрования, может быть любым
  resave: false, // Пересохранять ли куку при каждом запросе
  saveUninitialized: false, // Создавать ли сессию без инициализации ключей в req.session
  cookie: {
    maxAge: 1000 * 60 * 60 * 12, // Срок истечения годности куки в миллисекундах
    httpOnly: true, // Серверная установка и удаление куки, по умолчанию true
  },
};
app.use(session(sessionConfig));

app.use((req, res, next) => {
  res.locals.path = req.originalUrl;
  res.locals.user = req.session.user;
  next();
});

app.use('/auth', authRouter);

app.get('/', (req, res) => {
  res.render('Layout');
});

app.get('/auth', (req, res) => {
  res.render('Layout');
});

app.get('/reg', (req, res) => {
  res.render('Layout');
});

app.use(isAuth); // миделвара для проверки регистрацииставить на этапе где нужна поверка!

app.get('/users', async (req, res) => {
  const users = await User.findAll();
  const initState = { users };
  res.render('Layout', initState);
});

app.listen(PORT, () => console.log(`Server is started on port ${PORT}`));

```
- [X] Создаём папку `utils` и в ней файл `jsxRender.js`

```Javascript 
import React from 'react';
import { renderToString } from 'react-dom/server';// Подключение метода для перевода React компонента в строку
import Layout from '../components/Layout';

export default function jsxRender(_, initState, cb) {
  // Создаём реакт-компонент, содержащий разметку страницы
  const layout = React.createElement(Layout, { initState });
  const html = renderToString(layout); // Переводим этот компонент в строку
  // Возвращаем callback с html страницей
  return cb(null, `<!DOCTYPE html>${html}`);
}
```

## :pushpin: SRC/COMPONENTS
>Зачем это нужно?
>>App.jsx – это точка входа в наше приложение на фронтэнде
>>>Layout.jsx – разметка html документа
>>>>index.jsx позволяет наполнить скриптами статичный html с помощью метода hydrateRoot()
>>В будущем все React компоненты будут в данной папке src

### App.jsx
- [X] Создаём папку `src/components` и в ней файл `App.jsx`
Разворачиваем сниппет `rfc` (react functional component):
```Javascript
import React from 'react';
import { Route, Routes } from 'react-router-dom';
import Auth from './Auth';
import Home from './Home';
import Navbar from './Navbar';
import Reg from './Reg';
import Users from './Users';

export default function App({ user, users }) {
  console.log(user);
  return (
    <div className="container">
      <Navbar user={user} />
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/auth" element={<Auth />} />
        <Route path="/reg" element={<Reg />} />
        <Route path="/users" element={<Users users={users} />} />
      </Routes>
    </div>
  );
}
```

### Layout.jsx
- [X] В папке `src/components` создаём файл `Layout.jsx`
- [X] Разворачиваем сниппет `rfc` (react functional component)
- [X] Наполняем стандартной разметкой html-страницы
- [X] В теге body создаём 
`<div id=”root>`, внутрь которого помещается `<App />`
Подключаем скрипты    `app.js и vendor.js`

```Javascript
import React from 'react';
import { StaticRouter } from 'react-router-dom/server';
import App from './App';

export default function Layout({ initState }) {
  return (
    <html lang="en">
      <head>
        <meta charSet="UTF-8" />
        <meta httpEquiv="X-UA-Compatible" content="IE=edge" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <script
          type="text/javascript"
          dangerouslySetInnerHTML={{
            __html: `window.initState=${JSON.stringify(initState)}`,
          }}
        />
        <script defer src="/app.js" />
        <script defer src="/vendor.js" />
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-Zenh87qX5JnK2Jl0vWa8Ck2rdkQ2Bzep5IDxbcnCeuOxjzrPF/et3URy9Bv1WTRi" crossOrigin="anonymous" />
        <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-OERcA2EqjJCMA+/3y+gxIOqMEjwtxJY7qPCqsdltbNJuaOe923+mo//f6V8Qbsw3" crossOrigin="anonymous" />
        <title>Document</title>
      </head>
      <body>
        <div id="root">
          {console.log(initState)}
          <StaticRouter location={initState.path}>
            <App {...initState} />
          </StaticRouter>
        </div>
      </body>
    </html>
  );
}
```

## :pushpin: hydrateRoot
Если нужна гидратация, то:
- [X] В этой же папке создаём index.jsx
Задаём гидратацию:
```Javascript
import React from 'react';
import ReactDOMClient from 'react-dom/client';
import { BrowserRouter } from 'react-router-dom';
import App from './App';

ReactDOMClient.hydrateRoot(
  document.getElementById('root'),
  <BrowserRouter>
    <App {...window.initState} />
  </BrowserRouter>,
);

```
### /routes/authRouter.js
```Javascript
import express from 'express';
import { hash, compare } from 'bcrypt';
import { User } from '../../db/models';

const router = express.Router();

router.post('/', async (req, res) => {
  const { email, password } = req.body;
  if (!email || !password) return res.sendStatus(400);

  const user = await User.findOne({ where: { email } });

  if (!user) return res.sendStatus(400);

  const isPassValid = compare(password, user.password);

  if (!isPassValid) return res.sendStatus(400);

  req.session.user = { id: user.id, email: user.email };

  res.sendStatus(200);
});

router.post('/reg', async (req, res) => {
  const { email, password } = req.body;
  if (!email || !password) return res.sendStatus(400);

  const hashPassword = await hash(password, 10);

  const [user, isCreated] = await User.findOrCreate({
    where: { email },
    defaults: { email, password: hashPassword },
  });

  if (!isCreated) return res.sendStatus(400);

  req.session.user = { id: user.id, email: user.email };

  res.sendStatus(200);
});

router.get('/logout', (req, res) => {
  res.clearCookie('user_sid'); // Удалить куку
  req.session.destroy(); // Завершить сессию
  res.redirect('/');
});

export default router;
```


### /midlewares/isAuth.js
```Javascript
const isAuth = (req, res, next) => {
  if (!req.session.user) return res.redirect('/');
  next();
};

export default isAuth;
```
### Auth.jsx
```Javascript
import React from 'react';

function Auth() {
  const submitHandler = async (e) => {
    e.preventDefault();

    const response = await fetch('/auth/', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(Object.fromEntries(new FormData(e.target))),
    });

    if (response.ok) {
      window.location.href = '/';
    }
  };

  return (
    <div>
      <h1>Auth</h1>
      <form onSubmit={submitHandler}>
        <div className="mb-3">
          <label htmlFor="exampleInputEmail1" className="form-label">Почта</label>
          <input name="email" type="email" className="form-control" id="exampleInputEmail1" aria-describedby="emailHelp" />
        </div>
        <div className="mb-3">
          <label htmlFor="exampleInputPassword1" className="form-label">Пароль</label>
          <input name="password" type="password" className="form-control" id="exampleInputPassword1" />
        </div>
        <button type="submit" className="btn btn-primary">Авторизация</button>
      </form>
    </div>
  );
}

export default Auth;
```
### Reg.jsx
```Javascript
import React from 'react';

function Reg() {
  const submitHandler = async (e) => {
    e.preventDefault();

    const response = await fetch('/auth/reg/', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(Object.fromEntries(new FormData(e.target))),
    });

    if (response.ok) {
      window.location.href = '/';
    }
  };

  return (
    <div>
      <h1>Reg</h1>
      <form onSubmit={(e) => submitHandler(e)}>
        <div className="mb-3">
          <label htmlFor="exampleInputEmail1" className="form-label">Почта</label>
          <input name="email" type="email" className="form-control" id="exampleInputEmail1" aria-describedby="emailHelp" />
        </div>
        <div className="mb-3">
          <label htmlFor="exampleInputPassword1" className="form-label">Пароль</label>
          <input name="password" type="password" className="form-control" id="exampleInputPassword1" />
        </div>
        <button type="submit" className="btn btn-primary">Регистрация</button>
      </form>
    </div>
  );
}

export default Reg;
```


## :pushpin: FETCH API
### Пример (frontend)
```Javascript
import React from 'react';

export default function App() {
  const handler = () => {
    fetch('https://jsonplaceholder.typicode.com/todos/1')
      .then((response) => response.json())
      .then((data) => console.log(data));
  };
  return (
    <div>
      <button type="button" onClick={handler}>
        Click me!
      </button>
    </div>
  );
}
```

### AJAX отправка формы
Для AJAX отправки формы нужно
- [X] Добавить обработчик события отправки формы (атрибуты `onSubmit, onClick`)
- [X] Предотвратить стандартное поведение, перезагружающее страницу: `event.preventDefault()`
- [X] Сделать самостоятельный сетевой запрос (например, fetch)

####  пример AJAX отправки 
Frontend

```Javascript

import React from 'react';

export default function App() {
  const handler = (e) => {
    e.preventDefault();
    fetch('/messages', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(Object.fromEntries(new FormData(e.target))),
    });
  };
  return (
    <form onSubmit={handler}>
      <input name="title" />
      <input name="message" />
      <button type="submit">Send</button>
    </form>
  );
}
```
Backend
```Javascript
import express from 'express';
import template from './src/template';

const PORT = 3000;
const app = express();

app.use(express.static('public'));
app.use(express.urlencoded({ extended: true }));
app.use(express.json());

app.get('/', (req, res) => res.send(template({ path: req.originalUrl })));

app.post('/messages', (req, res) => {
  console.log(req.body); // Здесь находятся данные формы
  res.sendStatus(200);
});

app.listen(PORT, () => console.log(`App has started on port ${PORT}`));
```

## Cookie, sessions
####Backend

- [X] Чтобы использовать express session, установите `npm i express-session session-file-store`
- [X] В server.js создайте конфигурационный объект, подключите сессии и их хранилище:
```Javascript
import session from 'express-session';
import store from 'session-file-store';
//
const FileStore = store(session);
//
const sessionConfig = {
  name: 'user_sid',         // Имя куки для хранения id сессии. По умолчанию - connect.sid
  secret: process.env.SESSION_SECRET ?? 'test', // Секретное слово для шифрования
  resave: true,         // Пересохранять ли куку при каждом запросе
  store: new FileStore(),
  saveUninitialized: false,     // Создавать ли сессию без инициализации ключей в req.session
  cookie: {
    maxAge: 1000 * 60 * 60 * 12, // Срок истечения годности куки в миллисекундах
    httpOnly: true,         // Серверная установка и удаление куки, по умолчанию true
  },
};
//
app.use(session(sessionConfig));
```
- [X] разбиваем на роуты `routes/authRouter.js
```Javascript 
// РЕГИСТРАЦИЯ
router.post('/reg', async (req, res) => {
  const { userName, email, password } = req.body; // получаем данные от пользователя при регистрации
  // Проверяем что пришли все данные вместе и ничего не пришло пустым
  // эту проверку делать на Бэке, на фронте опасно
  if (!userName || !email || !password) {
    return res.sendStatus(400);
  } // если что то не пришло отправляем 400

  // ------
  // перед тем как зарегистрировать пользователя хэшируем pass
  // для этого импортируем библиотеку bcrypt для хэширования пароля ( npm i bcrypt )
  const hashPassword = await hash(password, 10);
  // регистрируем пользователя в БД и проверяем есть ли он в БД
  try {
    const [newUser, isCreated] = await User.findOrCreate({
      where: { email },
      defaults: { email, password: hashPassword },
    });
    if (!isCreated) return res.sendStatus(400);
    // сразу же авторизируем пользователя
    req.session.user = { id: newUser.id, email: newUser.email };
    // отправляем на фронт данные юзера
    // res.json({ id: newUser.id, userName: newUser.userName, email: newUser.email });
    return res.sendStatus(200);
  } catch (err) {
    console.error(err);
  }
  return res.sendStatus(200);
});
router.get('/logout', (req, res) => {
  res.clearCookie('user_sid'); // Удалить куку
  req.session.destroy(); // Завершить сессию
  res.redirect('/');
});
```

// РЕГИСТРАЦИЯ ФРОНТ
- [X] На форму навешиваем `onSubmit={handleSubmit}` ппрописываем у инпутов неймы  и пишем хендлер
```Javascript 
 const handleSubmit = async (e) => {
    e.preventDefault();
    const response = await fetch('/auth/reg/', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(Object.fromEntries(new FormData(e.target))),
    });

    if (response.ok) {
      window.location.href = '/';
    }
  };
```

## SEQUELIZE :memo:

###  .sequelizerc
>Sequelize позволяет управлять базой данных с помощью Javascript.

- [X] В корне проекта создаём файл `.sequelizerc`
Вставляем туда:


```Javascript
const path = require('path');

module.exports = {
  'config': path.resolve('db', 'database.js'),
  'models-path': path.resolve('db', 'models'),
  'seeders-path': path.resolve('db', 'seeders'),
  'migrations-path': path.resolve('db', 'migrations')
};

```
### DB
- [X] Через консоль заходим в postgres и создаём необходимую для задачи базу данных
`psql postgres
CREATE DATABASE dbname OWNER admin;
quit
`

### database.json

- [X] Инициализируем sequelize
`npx sequelize-cli init`
Находим файл src/db/config/database.json  и конфигурируем его

### model
- [X] Создаём модели по одной (начиная с независимых). 
*Например, создаём модель User (пишем в единственном числе и с заглавной буквы):*
`npx sequelize-cli model:generate --name User --attributes name:string`
- [X] Прописываем зависимости references внутри миграций, belongsTo и hasMany в моделях
- [X] Накатываем миграции
`npx sequelize-cli db:migrate`

### seeders
- [X] Создаём сидеры по одному (начиная с независимых):
`npx sequelize-cli seed:generate --name users`
- [X] Внутри пользуемся методом bulkInsert
- [X] Накатываем сиды
```npx sequelize-cli db:seed:all```

### .env
- [X] Создаём файл `copyEnv`
```
DB_USER=
DB_PASS=
DB_NAME=

PORT=
```
- [X] Создаём файл `.env`
``` DB_USER=elbrus
DB_PASS=123
DB_NAME=elbrus
DB_HOST=127.0.0.1

PORT=3000
```
### db/database.js
```Javascript
require('dotenv').config();

module.exports = {
  development: {
    username: process.env.DB_USER,
    password: process.env.DB_PASS,
    database: process.env.DB_NAME,
    host: process.env.DB_HOST,
    dialect: 'postgres',
  },
  test: {
    username: 'root',
    password: null,
    database: 'database_test',
    host: '127.0.0.1',
    dialect: 'mysql',
  },
  production: {
    username: process.env.DB_USER,
    password: process.env.DB_PASS,
    database: process.env.DB_NAME,
    host: process.env.DB_HOST,
    dialect: 'postgres',
    dialectOptions: {
      ssl: {
        rejectUnauthorized: false,
      },
    },
  },
};
```













#### Пример как засидить БД

```Javascript
const { faker } = require('@faker-js/faker');
/** @type {import('sequelize-cli').Migration} */
module.exports = {
  async up(queryInterface, Sequelize) {
    const roleArr = [];
    for (let i = 0; i < 10; i++) {
      roleArr.push(
        {
          name: faker.name.jobTitle(),
          description: faker.hacker.phrase(),
          createdAt: new Date(),
          updatedAt: new Date(),
        },
      );
    }

    await queryInterface.bulkInsert('Roles', roleArr, {});
  },


```
