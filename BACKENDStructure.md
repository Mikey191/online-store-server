# Структура Backend
## Используемые технологии:
1. node.js - платформа для яп JS
2. express - создание маршрутов для http запросов
3. postgresql - база данных для хранения информации
4. sequelize - ORM для работы с postgres
5. jwt - токен для аутентификации и авторизации

## Структура REST API для online-store:
1. Получение информации о продуктах
2. Добавление продуктов в корзину
3. Оформление заказа
4. Получение информации о заказах
5. Аутентификация и авторизация

## Библиотеки для разработки серверной части приложения:
1. npm install express
2. npm install pg pg-hstore
3. npm install sequelize
4. npm install cors
5. npm install dotenv
6. npm install -D nodemon

## Создание сервера:
Создаем файл для переменных окружения .env
```
PORT=5000 // Порт на котором будет работать сервер
DB_NAME=<online-store> // Название БД
DB_USER=<postgres> // Имя пользователя
DB_PASSWORD=<Password_postgres> // Пароль пользователя
DB_HOST=<localhost> // Хост для режима разработки
DB_PORT=<5432> // Порт который указывали при создании БД
SECRET_KEY=random_str_key123
```
Файл index.js
```javascript
require("dotenv").config(); // для загрузки переменных окружения из файла .env
const express = require('express'); // импорт express
conxt app = express(); //Объект для запуска приложения
const PORT = process.env.PORT || 5000; //переменная для хранения порта приложения
app.listen(PORT, () => console.log(`Server started on port ${PORT}`)); // запускаем сервер
```

## Подключение к БД:
Создание файла db.js
```javascript
const { Sequelize } = require("sequelize"); // импорт Sequelize
module.exports = new Sequelize( // создаем объект класса Sequelize
 process.env.DB_NAME,
  process.env.DB_USER,
  process.env.DB_PASSWORD,
  {
    dialect: "postgres",
    host: process.env.DB_HOST,
    port: process.env.DB_PORT,
  }
); 
```
Файл index.js
```javascript
require("dotenv").config(); // для загрузки переменных окружения из файла .env
const express = require('express'); // импорт express
const sequelize = require("./db"); // импортируем созданный объект типа Sequelize
conxt app = express(); // Объект для запуска приложения

const PORT = process.env.PORT || 5000; //переменная для хранения порта приложения

const start = async () => { // функция для старта приложения
  try {
    await sequelize.authenticate(); // для проверки соединения с базой данных
    await sequelize.sync() // создает таблицы в базе данных на основе определений моделей, если они еще не существуют
    app.listen(PORT, () => console.log(`Server started on port ${PORT}`)); // запускаем сервер
  } catch (error) {
    console.log(error);
  }
};

start();
```

## Модели данных и связи между ними.
Создание папки models и файла models.js
```javascript
const sequelize = require("./db"); // импортируем созданный объект типа Sequelize
const { DataTypes } = require("sequelize"); // содержит различные типы данных, которые могут быть использованы при определении моделей
// модель пользователя User
const User = sequelize.define("user", { 
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
  email: { type: DataTypes.STRING, unique: true },
  password: { type: DataTypes.STRING },
  role: { type: DataTypes.STRING, defaultValue: "USER" },
});
// модель корзины Basket
const Basket = sequelize.define("basket", { 
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
});
// модель BasketDevice
const BasketDevice = sequelize.define("basket_device", { 
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
});
// модель Device
const Device = sequelize.define("device", {
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
  name: { type: DataTypes.STRING, unique: true, allowNull: false },
  price: { type: DataTypes.FLOAT, allowNull: false },
  rating: { type: DataTypes.FLOAT, defaultValue: 0 },
  img: { type: DataTypes.STRING, allowNull: false },
});
// модель Type
const Type = sequelize.define("type", {
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
  name: { type: DataTypes.STRING, unique: true, allowNull: false },
});
// модель Brand
const Brand = sequelize.define("brand", {
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
  name: { type: DataTypes.STRING, unique: true, allowNull: false },
});
// модель Rating
const Rating = sequelize.define("rating", {
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
  rate: { type: DataTypes.INTEGER, allowNull: false },
});
// модель DeviceInfo
const DeviceInfo = sequelize.define("device_info", {
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
  title: { type: DataTypes.STRING, allowNull: false },
  description: { type: DataTypes.STRING, allowNull: false },
});
// модель промежуточной таблицы для связи many-to-many Type и Brand
const TypeBrand = sequelize.define("type_brand", {
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
});

// связи таблиц
User.hasOne(Basket); // связь один к одному
Basket.belongsTo(User);

User.hasMany(Rating); // связь один ко многим
Rating.belongsTo(User);

Basket.hasMany(BasketDevice); // связь один ко многим
BasketDevice.belongsTo(Basket);

Type.hasMany(Device); // связь один ко многим
Device.belongsTo(Type);

Brand.hasMany(Device); // связь один ко многим
Device.belongsTo(Brand);

Device.hasMany(Rating); // связь один ко многим
Rating.belongsTo(Device);

Device.hasMany(BasketDevice); // связь один ко многим
BasketDevice.belongsTo(Device);

Device.hasMany(DeviceInfo, {as: "info"}); // связь один ко многим
DeviceInfo.belongsTo(Device);

Type.belongsToMany(Brand, { through: TypeBrand }); // связь много ко многим. для этой связи создавали промежуточную таблицу TypeBrand
Brand.belongsToMany(Type, { through: TypeBrand });

// экспорт созданных сущностей
module.exports = {
  User,
  Basket,
  BasketDevice,
  Device,
  Type,
  Brand,
  Rating,
  TypeBrand,
  DeviceInfo,
};
```
Файл index.js
```javascript
require("dotenv").config(); // для загрузки переменных окружения из файла .env
const express = require('express'); // импорт express
const sequelize = require("./db"); // импортируем созданный объект типа Sequelize
const cors = require("cors") // импорт cors

const PORT = process.env.PORT || 5000; //переменная для хранения порта приложения

conxt app = express(); // объект для запуска приложения
app.use(cors()) // подключаем CORS
app.use(express.json()) // для парсинга json формата
app.get('/', (req, res) => {res.status(200).json({message: "WORKING"})})

const start = async () => { // функция для старта приложения
  try {
    await sequelize.authenticate(); // для проверки соединения с базой данных
    await sequelize.sync() // создает таблицы в базе данных на основе определений моделей, если они еще не существуют
    app.listen(PORT, () => console.log(`Server started on port ${PORT}`)); // запускаем сервер
  } catch (error) {
    console.log(error);
  }
};

start();
```