# Структура Backend
## Для быстрого доступа
1. Создание сервера
2. Подключение к БД
3. Модели данных и связи между ними
4. Маршруты приложения
5. Универсальная обработка ошибки
6. Добавление объектов Type и Brand в Базу Данных.
7. Получение объектов Type и Brand из Базы Данных.
8. Работа с файлами (загружаем картинки)

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
1. `npm install express`
2. `npm install pg pg-hstore`
3. `npm install sequelize`
4. `npm install cors`
5. `npm install dotenv`
6. `npm install -D nodemon`
7. `npm install jsonwebtoken`
8. `npm install bcrypt` - **для хеширования паролей**
9. `npm install express-fileupload` - **для хеширования паролей**
10. `npm install uuid` - **для генерации уникальных идентификаторов**

## 1. Создание сервера:
### Создаем файл для переменных окружения .env
```
PORT=5000 // Порт на котором будет работать сервер
DB_NAME=<online-store> // Название БД
DB_USER=<postgres> // Имя пользователя
DB_PASSWORD=<Password_postgres> // Пароль пользователя
DB_HOST=<localhost> // Хост для режима разработки
DB_PORT=<5432> // Порт который указывали при создании БД
SECRET_KEY=random_str_key123
```
Файл `index.js`
```javascript
require("dotenv").config(); // для загрузки переменных окружения из файла .env
const express = require('express'); // импорт express
conxt app = express(); //Объект для запуска приложения
const PORT = process.env.PORT || 5000; //переменная для хранения порта приложения
app.listen(PORT, () => console.log(`Server started on port ${PORT}`)); // запускаем сервер
```

## 2. Подключение к БД:
### Создание файла `db.js`
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
### Файл `index.js`
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

## 3. Модели данных и связи между ними.
### Создание папки `models` и файла `models.js`
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
### Файл `index.js`
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

## 4. Маршруты приложения
Создание папки `routes` и файла `index.js`  
Создаем для каждой сущности файлы с маршрутами `userRoutes.js`, `brandRoutes.js`, `typeRoutes.js`, `deviceRoutes.js`  
Создаем для реализации логики каждого маршрута папку `controllers` и файлы `userController.js`, `brandController.js`, `typeController.js`, `deviceController.js`  

### Файлы userController.js, brandController.js, typeController.js, deviceController.js:
#### Файл `userController.js`
```javascript
class UserController {
  async registration(req, res) {}
  async login(req, res) {}
  async check(req, res) {}
}

module.exports = new UserController();
```

#### Файл `brandController.js`
```javascript
class BrandController {
  async create(req, res) {}
  async getAll(req, res) {}
}

module.exports = new BrandController();
```

#### Файл `typeController.js`
```javascript
class TypeController {
  async create(req, res) {}
  async getAll(req, res) {}
}

module.exports = new TypeController();
```

#### Файл `deviceController.js`
```javascript
class DeviceController {
  async create(req, res) {}
  async getAll(req, res) {}
  async getOne(req, res) {}
}

module.exports = new DeviceController();
```

### Файлы userRoutes.js, brandRoutes.js, typeRoutes.js, deviceRoutes.js:
#### Файл `userRoutes.js`
```javascript
const Router = require("express");
const router = new Router();
const userController = require("../controllers/userController");
const authMiddleware = require("../middleware/authMiddleware");

router.post("/registration", userController.registration);
router.post("/login", userController.login);
router.get("/auth", userController.check);

module.exports = router;
```

#### Файл `brandRoutes.js`
```javascript
const Router = require("express");
const router = new Router();
const brandController = require("../controllers/brandController");

router.post('/', brandController.create)
router.get('/', brandController.getAll)

module.exports = router;
```

#### Файл `typeRoutes.js`
```javascript
const Router = require("express");
const router = new Router();
const typeController = require("../controllers/typeController");
const checkRole = require("../middleware/checkRoleMiddleware");

router.post("/", typeController.create);
router.get("/", typeController.getAll);

module.exports = router;
```

#### Файл `deviceRoutes.js`
```javascript
const Router = require("express");
const router = new Router();
const deviceController = require("../controllers/deviceController")

router.post("/", deviceController.create);
router.get("/", deviceController.getAll);
router.get("/:id", deviceController.getOne);

module.exports = router;
```

### Файл `routes/index.js`:
```javascript
const Router = require('express'); // импортируем класс Роутер
const router = new Router(); // создаем объект этого класса
const deviceRouter = require("./deviceRouter"); // роутер device
const userRouter = require("./userRouter"); // роутер user
const brandRouter = require("./brandRouter"); // роутер brand
const typeRouter = require("./typeRouter"); // роутер type

router.use("/user", userRouter); // маршрут для user
router.use("/type", typeRouter); // маршрут для type
router.use("/brand", brandRouter); // маршрут для brand
router.use("/device", deviceRouter); // маршрут для device

module.exports = router; // экспортируем роутер
```

### Файл `index.js`
```javascript
require("dotenv").config(); // для загрузки переменных окружения из файла .env
const express = require('express'); // импорт express
const sequelize = require("./db"); // импортируем созданный объект типа Sequelize
const cors = require("cors") // импорт cors
const router = require("./routes/index"); // импортируем маршруты

const PORT = process.env.PORT || 5000; //переменная для хранения порта приложения

conxt app = express(); // объект для запуска приложения
app.use(cors()) // подключаем CORS
app.use(express.json()) // для парсинга json формата
app.use("/api", router); // для использования маршрутов в приложении

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

## 5. Универсальная обработка ошибок
При вызове функции check, если пользователь не указал параметр id, то мы прокидываем ошибку.  
### Создание папки `error` и файла `ApiError.js`
```javascript
class ApiError extends Error { // универсальный handler для отлова ошибок наследуемый от Error
  constructor(status, message) { //конструктор принимает стутс код и сообщение об ошибки
    super();
    this.status = status; // присваиваем полученный статус
    this.message = message; // присваиваем полученное сообщение об ошибки
  }
  static badRequest(message) { // функция для ошибки со статус-кодом 404
    return new ApiError(404, message);
  }
  static internal(message) { // функция для ошибки со статус-кодом 500
    return new ApiError(500, message);
  }
  static forbidden(message) { // функция для ошибки со статус-кодом 403
    return new ApiError(403, message);
  }
}

module.exports = ApiError;
```

### Создание папки `middleware` и файла `ErrorHandlingMiddleware.js`
```javascript
const ApiError = require("../error/ApiError"); // импортируем класс для обработки ошибок

module.exports = function (err, req, res, next) { // аргументы - ошибка, запрос, ответ, функция next для вызова следующего в цепочки middleware
  if (err instanceof ApiError) { // проверяем принадлежит ли сообщение об ошибки классу ApiError
    return res.status(err.status).json({ message: err.message }); // на клиент отправляем сообщение об ошибки
  }
  return res.status(500).json({ message: "Непредвиденная ошибка!" }); // если ошибка не принадлежит обработанным ошибкам отправляем статус-код 500 и сообщение
  // внутри этого хендлера мы не вызываем next(), потому что он является замыкающим в цепочке middleware
};
```

### Файл `index.js`
```javascript
require("dotenv").config(); // для загрузки переменных окружения из файла .env
const express = require('express'); // импорт express
const sequelize = require("./db"); // импортируем созданный объект типа Sequelize
const cors = require("cors") // импорт cors
const router = require("./routes/index"); // импортируем маршруты
const errorHandler = require("./middleware/ErrorHandingMiddleware"); // импортируем хендлер ошибок

const PORT = process.env.PORT || 5000; //переменная для хранения порта приложения

conxt app = express(); // объект для запуска приложения
app.use(cors()) // подключаем CORS
app.use(express.json()) // для парсинга json формата
app.use("/api", router); // для использования маршрутов в приложении


// Обработка ошибок должна быть в конце
app.use(errorHandler);
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

## 6. Добавление объектов Type и Brand в Базу Данных.
### Файл `TypeController.js`
```javascript
const { Type } = require("../models/models"); // импортируем модель Type
const ApiError = require("../error/ApiError"); // импортируем хендлер ошибок

class TypeController {
  async create(req, res, next) {
    try {
      const { name } = req.body; // Название типа из тела запроса
      const type = await Type.create({ name }); // Создаем тип
      return res.json(type); 
    } catch (error) {
      next(ApiError.badRequest(error.message)); // Возвращаем ошибку
    }
  }

  async getAll(req, res) {}
}

module.exports = new TypeController();
```
### Файл `brandController.js`
```javascript
const { Brand } = require("../models/models"); // импортируем модель Brand
const ApiError = require("../error/ApiError"); // импортируем хендлер ошибок

class BrandController {
  async create(req, res, next) {
    try {
      const { name } = req.body; // Название бренда из тела запроса
      const brand = await Brand.create({ name }); // Создаем бренд
      return res.json(brand);
    } catch (error) {
      next(ApiError.badRequest(error.message)); // Возвращаем ошибку
    }
  }
  async getAll(req, res) {}
}

module.exports = new BrandController();
```
## 7. Получение объектов Type и Brand из Базы Данных.
### Файл `TypeController.js`
```javascript
const { Type } = require("../models/models");
const ApiError = require("../error/ApiError");

class TypeController {
  async create(req, res, next) {
    try {
      const { name } = req.body;
      const type = await Type.create({ name });
      return res.json(type);
    } catch (error) {
      next(ApiError.badRequest(error.message));
    }
  }
  async getAll(req, res) {
    const types = await Type.findAll(); // возвращаем все существующие записи типа
    return res.json(types); // на клиент возвращаем весь массив объектов
  }
}

module.exports = new TypeController();
```

### Файл `brandController.js`
```javascript
const { Brand } = require("../models/models");
const ApiError = require("../error/ApiError");

class BrandController {
  async create(req, res, next) {
    try {
      const { name } = req.body;
      const brand = await Brand.create({ name });
      return res.json(brand);
    } catch (error) {
      next(ApiError.badRequest(error.message));
    }
  }
  async getAll(req, res) {
    const brands = await Brand.findAll(); // возвращаем все существующие записи бренда
    return res.json(brands); // на клиент возвращаем весь массив объектов
  }
}

module.exports = new BrandController();
```

## 8. Работа с файлами (загружаем картинки)
При создании device нам нужно будет подгружать картинку к каждому device.  
Библиотека для работы с картинками - `express-fileupload`.  
#### Зарегестрировать пакет в `index.js`:
```javascript
require("dotenv").config(); // для загрузки переменных окружения из файла .env
const express = require('express'); // импорт express
const sequelize = require("./db"); // импортируем созданный объект типа Sequelize
const cors = require("cors") // импорт cors
const router = require("./routes/index"); // импортируем маршруты
const errorHandler = require("./middleware/ErrorHandingMiddleware"); // импортируем хендлер ошибок
const fileUpload = require("express-fileupload"); // импортируем fileupload

const PORT = process.env.PORT || 5000; //переменная для хранения порта приложения

conxt app = express(); // объект для запуска приложения
app.use(cors()) // подключаем CORS
app.use(express.json()) // для парсинга json формата
app.use("/api", router); // для использования маршрутов в приложении
app.use(fileUpload({})); // регестрируем fileUpload


// Обработка ошибок должна быть в конце
app.use(errorHandler);
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
### Имя для картинки.
При сохранении картинки на сервере нужно будет генерировать имя для картинки.  
Для генерации `имени` нужна библиотека `npm install uuid`.  
Для хранения файлов создадим папку `static`
### Файл `deviceController.js`
```javascript
const uuid = require("uuid");
const { Device, DeviceInfo } = require("../models/models");
const ApiError = require("../error/ApiError");

class DeviceController {
  async create(req, res, next) {
    try {
      let { name, price, brandId, typeId, info } = req.body; // параметры для девайса получаемые из тела запроса
      const { img } = req.files; // картинка для каждого девайса
      let fileName = uuid.v4() + ".jpg"; // генерируем название файла
      img.mv(path.resolve(__dirname, "..", "static", fileName)); // перемещаем файл в папке static
      // создаем девайс в Базе Данных
      const device = await Device.create({
        name,
        price,
        brandId,
        typeId,
        img: fileName,
      });
      return res.json(device); // возвращаем полученный девайс на клиент
    } catch (error) {
      next(ApiError.badRequest(error.message));
    }
  }
  async getAll(req, res) {}
  async getOne(req, res) {}
}

module.exports = new DeviceController();
```
### Раздача СТАТИЧЕСКИХ файлом, которые находятся на нашем сервере.
Для раздачи статических файлов необходимо указать серверу явно.  
Файлы будут раздаваться из папки `static`.
### Файл `index.js` 
```javascript
require("dotenv").config(); // для загрузки переменных окружения из файла .env
const express = require('express'); // импорт express
const sequelize = require("./db"); // импортируем созданный объект типа Sequelize
const cors = require("cors") // импорт cors
const router = require("./routes/index"); // импортируем маршруты
const errorHandler = require("./middleware/ErrorHandingMiddleware"); // импортируем хендлер ошибок
const fileUpload = require("express-fileupload"); // импортируем fileupload
const path = require("path") // импортируем библиотеку для универсального указания пути

const PORT = process.env.PORT || 5000; //переменная для хранения порта приложения

conxt app = express(); // объект для запуска приложения
app.use(cors()) // подключаем CORS
app.use(express.json()) // для парсинга json формата
app.use(express.static(path.resolve(__dirname, "static"))) // явно указываем из какой папки сервер будет раздовать статику
app.use(fileUpload({})); // регестрируем fileUpload
app.use("/api", router); // для использования маршрутов в приложении


// Обработка ошибок должна быть в конце
app.use(errorHandler);
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