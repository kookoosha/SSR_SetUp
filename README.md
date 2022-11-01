
## ИНИЦИАЛИЗАЦИЯ ПРОЕКТА - GIT


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


## BABEL
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
## WEBPACK
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

## SRC / SERVER
>Все данные проекта будут находиться в папке src. Файл server.js будет являться стартовой точкой запуска express.
Скрипты в package.json добавляем, чтобы можно было разделить запуск webpack и express

- [X] В корне проекта создаём папку src
- [X] Добавляем туда пустой файл `server.js`
- [X] Добавляем в `package.json` скрипты запуска сервера и сборки webpack:
```Javascript 
 "scripts": {
    "webpack": "webpack -wd eval-source-map",
    "dev": "babel-node src/server.js",
  },

```
- [X] В папке src/routes -> разбиваем логику роутов на разные файлы
-
##### Пример server.js
```Javascript 
// Подключение библиотеки express
import express from 'express';
// Подключаем переменную path для того что бы путь был найден при запуске из другого места
import path from 'path';
// Импорт кастомного рендера jsx-файлов
import jsxRender from './utils/jsxRender';

// Создание приложения express
const app = express();
// Регистрируем движок для рендера
app.engine('jsx', jsxRender);
app.set('view engine', 'jsx');
// Указываем путь до компонентов
app.set('views', path.join(__dirname, 'components'));
// Изначальное значение счетчика
app.locals.count = 0;
// Подключение middleware, который парсит JSON от клиента
app.use(express.json());
// Подключение middleware, который отдаёт клиенту файлы из папки
app.use(express.static(path.join(__dirname, 'public')));

// Роут, отвечающий на запрос GET /
app.get('/', (req, res) => {
  // Передаем в Layout count который должен отобразиться в App.jsx при первом открытии страницы
  const initState = { count: app.locals.count };
  res.render('Layout', initState)
});

// Роут, отвечающий на запрос GET /next
app.get('/next', (req, res) => {
  app.locals.count += 1;
  return res.json({
    count: app.locals.count,
  });
});

// Запуск сервера по порту 3000
app.listen(3000, () => {
  console.log('server start');
});

```
- [X] Создаём папку `utils` и в ней файл `jsxRender.js`

```Javascript 
import React from 'react';
// Подключение метода для перевода React компонента в строку
import { renderToString } from 'react-dom/server';
import Layout from '../components/Layout';

export default function jsxRender(pathToFile, initState, cb) {
  // Создаём реакт-компонент, содержащий разметку страницы
  const layout = React.createElement(Layout, { initState });
  // Переводим этот компонент в строку
  const html = renderToString(layout);
  // Возвращаем callback с html страницей
  return cb(null, `<!DOCTYPE html>${html}`);
}
```

## SRC/COMPONENTS
>Зачем это нужно?
>>App.jsx – это точка входа в наше приложение на фронтэнде
>>>Layout.jsx – разметка html документа
>>>>index.jsx позволяет наполнить скриптами статичный html с помощью метода hydrateRoot()
>>В будущем все React компоненты будут в данной папке src

### App.jsx
- [X] Создаём папку `src/components` и в ней файл `App.jsx`
Разворачиваем сниппет `rfc` (react functional component):
```Javascript
import React, { useState } from 'react';
/*
получаем count из пропсов переданных в Layout в файле server.js
*/
export default function App({ count }) {
// Объявляем состояние которое будет хранить состояние счетчика
  const [appCount, setAppCount] = useState(count);

  // Добавляем функцию которая будет выполнять fetch запрос при нажатии на кнопку
  const counterHandler = () => {
    fetch('/next')
      .then((res) => res.json())
      .then((res) => setAppCount(res.count));
  };

  return (
    <>
      <h1>Кусочек</h1>
      <span>{appCount}</span>
      <button
        type="button"
        onClick={counterHandler} // Создаем обработчик события click
      >
        Следующий
      </button>
    </>
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
import App from './App';

export default function Layout({ initState }) {
  return (
    <html lang="en">
      <head>
        <meta charSet="UTF-8" />
        <meta httpEquiv="X-UA-Compatible" content="IE=edge" />
        <meta name="viewport" content="width=device-width, height=device-height, initial-scale=1.0, user-scalable=yes" />
        {/* скрипт наполнения вспомогательнго объекта initState для работы гидратации */}
        <script
          type="text/javascript"
          dangerouslySetInnerHTML={{
            __html: `window.initState=${JSON.stringify(initState)}`,
          }}
        />
        {/* скрипты собранные через Webpack */}
        <script defer src="/app.js" />
        <script defer src="/vendor.js" />
        <title>Puzzle 420</title>
      </head>
      <body>
        <div id="root">
          <App {...initState} />
        </div>
      </body>
    </html>
  );
}
```

## hydrateRoot
Если нужна гидратация, то:
- [X] В этой же папке создаём index.jsx
Задаём гидратацию:
```Javascript
import React from 'react';
import ReactDOMClient from 'react-dom/client';
import App from './App';

ReactDOMClient.hydrateRoot(document.getElementById('root'), <App {...window.initState} />);
```
В Layout добавляем скрипт, который позволит передать initState в браузер:

```Javascript
<script type="text/javascript" dangerouslySetInnerHTML={{
__html: `window.initState=${JSON.stringify(initState)}`
          }} />
```
- [X] Если в реакте будет использоваться роутер, то импортируем react-router-dom и оборачиваем компонент <App /> в <BrowserRouter /> в файле index.jsx :

```Javascript
import { BrowserRouter } from 'react-router-dom';
import React from 'react';
import ReactDOMClient from 'react-dom/client';
import App from './App';

ReactDOMClient.hydrateRoot(
  document.getElementById('root'),
  <BrowserRouter>
    <App {...window.initState} />
  </BrowserRouter>,
);
```
- [X] А также в случае роутинга оборачиваем компонент <App /> в <StaticRouter /> внутри <Layout />
- [X] Указываем пропс location для работы hydrateRoot()


import { StaticRouter } from 'react-router-dom/server';
```Javascript
      <body>
        <div id="root">
          <StaticRouter location={initState.path}>
              <App {...initState} />
          </StaticRouter>
        </div>
      </body>
```








## SEQUELIZE 
###  .sequelizerc
>Sequelize позволяет управлять базой данных с помощью Javascript.

- [X] В корне проекта создаём файл `.sequelizerc`
Вставляем туда:


```Javascript
const path = require('path');

module.exports = {
  'config': path.resolve(''db', 'config', 'database.json'),
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
