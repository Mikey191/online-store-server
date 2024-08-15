# React & Node.js Online-Store-Server.

## Структура Backend
1. Создание сервера
2. Подключение к БД
3. Модели данных и связи между ними
4. Маршруты приложения
5. Универсальная обработка ошибки
6. Добавление объектов Type и Brand в Базу Данных.
7. Получение объектов Type и Brand из Базы Данных.
8. Работа с файлами (загружаем картинки)
9. Получение устройств, фильтрация, пагинация - постраничный вывод
10. Регистрация и авторизация. JWT токен. dcrypt.
11. middleware для Авторизации. Декодировать и проверять токен на валидность.
12. middleware для Проверки Роли Пользователя. 

## Дополнительная информация
1. Что такое request и response при работе с сайтами?
2. Что такое REST API?
3. Что такое ORM для реляционных баз данных?
4. Что такое JWT авторизация и JWT токен?
5. Что такое Bearer в токене?
6. Что такое CORS?
7. Что такое ВАЛИДАЦИЯ?
8. Использование super() при вызове конструктора в унаследованном классе.
9. Проектирование и написание middleware в Node.js
10. Методы объектов Sequelize
11. Метод OPTIONS в Express.

## Используемые технологии:
1. **node.js** - платформа для яп JS
2. **express** - создание маршрутов для http запросов
3. **postgresql** - база данных для хранения информации
4. **sequelize** - ORM для работы с postgres
5. **jwt** - токен для аутентификации и авторизации

## Библиотеки для разработки серверной части приложения:
1. `npm install express`
2. `npm install pg pg-hstore`
3. `npm install sequelize`
4. `npm install cors`
5. `npm install dotenv`
6. `npm install -D nodemon`
7. `npm install jsonwebtoken` - **для генерации JWT токена**
8. `npm install bcrypt` - **для хеширования паролей**
9. `npm install express-fileupload` - **для хеширования паролей**
10. `npm install uuid` - **для генерации уникальных идентификаторов**

## 1. Создание сервера:
### Создаем файл для переменных окружения `.env`:
```
PORT=5000 // Порт на котором будет работать сервер
DB_NAME=<online-store> // Название БД
DB_USER=<postgres> // Имя пользователя
DB_PASSWORD=<Password_postgres> // Пароль пользователя
DB_HOST=<localhost> // Хост для режима разработки
DB_PORT=<5432> // Порт который указывали при создании БД
SECRET_KEY=random_str_key123
```
### Файл `index.js`:
```javascript
require("dotenv").config(); // для загрузки переменных окружения из файла .env
const express = require('express'); // импорт express
conxt app = express(); //Объект для запуска приложения
const PORT = process.env.PORT || 5000; //переменная для хранения порта приложения
app.listen(PORT, () => console.log(`Server started on port ${PORT}`)); // запускаем сервер
```

## 2. Подключение к БД:
### Создание файла `db.js`:
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
### Файл `index.js`:
```javascript
require("dotenv").config(); 
const express = require('express'); 
const sequelize = require("./db"); // импортируем созданный объект типа Sequelize
conxt app = express(); 

const PORT = process.env.PORT || 5000; 

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

## 3. Модели данных и связи между ними:
### Создание папки `models` и файла `models.js`:
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

Device.hasMany(DeviceInfo); // связь один ко многим
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
### Файл `index.js`:
```javascript
require("dotenv").config(); 
const express = require('express'); 
const sequelize = require("./db"); 
const cors = require("cors") // импорт cors

const PORT = process.env.PORT || 5000; 

conxt app = express(); 
app.use(cors()) // подключаем CORS
app.use(express.json()) // для парсинга json формата
app.get('/', (req, res) => {res.status(200).json({message: "WORKING"})})

const start = async () => { 
  try {
    await sequelize.authenticate(); 
    await sequelize.sync() 
    app.listen(PORT, () => console.log(`Server started on port ${PORT}`));
  } catch (error) {
    console.log(error);
  }
};

start();
```

## 4. Маршруты приложения:
Создание папки `routes` и файла `index.js`  
Создаем для каждой сущности файлы с маршрутами `userRoutes.js`, `brandRoutes.js`, `typeRoutes.js`, `deviceRoutes.js`  
Создаем для реализации логики каждого маршрута папку `controllers` и файлы `userController.js`, `brandController.js`, `typeController.js`, `deviceController.js`  

### Файлы userController.js, brandController.js, typeController.js, deviceController.js:
#### Файл `userController.js`:
```javascript
class UserController {
  async registration(req, res) {}
  async login(req, res) {}
  async check(req, res) {}
}

module.exports = new UserController();
```

#### Файл `brandController.js`:
```javascript
class BrandController {
  async create(req, res) {}
  async getAll(req, res) {}
}

module.exports = new BrandController();
```

#### Файл `typeController.js`:
```javascript
class TypeController {
  async create(req, res) {}
  async getAll(req, res) {}
}

module.exports = new TypeController();
```

#### Файл `deviceController.js`:
```javascript
class DeviceController {
  async create(req, res) {}
  async getAll(req, res) {}
  async getOne(req, res) {}
}

module.exports = new DeviceController();
```

### Файлы userRoutes.js, brandRoutes.js, typeRoutes.js, deviceRoutes.js:
#### Файл `userRoutes.js`:
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

#### Файл `brandRoutes.js`:
```javascript
const Router = require("express");
const router = new Router();
const brandController = require("../controllers/brandController");

router.post('/', brandController.create)
router.get('/', brandController.getAll)

module.exports = router;
```

#### Файл `typeRoutes.js`:
```javascript
const Router = require("express");
const router = new Router();
const typeController = require("../controllers/typeController");
const checkRole = require("../middleware/checkRoleMiddleware");

router.post("/", typeController.create);
router.get("/", typeController.getAll);

module.exports = router;
```

#### Файл `deviceRoutes.js`:
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

### Файл `index.js`:
```javascript
require("dotenv").config(); 
const express = require('express'); 
const sequelize = require("./db"); 
const cors = require("cors") 
const router = require("./routes/index"); // импортируем маршруты

const PORT = process.env.PORT || 5000; 

conxt app = express(); 
app.use(cors()) 
app.use(express.json()) 
app.use("/api", router); // для использования маршрутов в приложении

const start = async () => { 
  try {
    await sequelize.authenticate(); 
    await sequelize.sync() 
    app.listen(PORT, () => console.log(`Server started on port ${PORT}`)); 
  } catch (error) {
    console.log(error);
  }
};

start();
```

## 5. Универсальная обработка ошибок.
При вызове функции check, если пользователь не указал параметр id, то мы прокидываем ошибку.  
### Создание папки `error` и файла `ApiError.js`:
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

### Создание папки `middleware` и файла `ErrorHandlingMiddleware.js`:
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

### Файл `index.js`:
```javascript
require("dotenv").config(); 
const express = require('express'); 
const sequelize = require("./db"); 
const cors = require("cors") 
const router = require("./routes/index"); 
const errorHandler = require("./middleware/ErrorHandingMiddleware"); // импортируем хендлер ошибок

const PORT = process.env.PORT || 5000; 

conxt app = express(); 
app.use(cors()) 
app.use(express.json()) 
app.use("/api", router); 


// Обработка ошибок должна быть в конце
app.use(errorHandler);
const start = async () => { 
  try {
    await sequelize.authenticate(); 
    await sequelize.sync() 
    app.listen(PORT, () => console.log(`Server started on port ${PORT}`)); 
  } catch (error) {
    console.log(error);
  }
};

start();
```

## 6. Добавление объектов Type и Brand в Базу Данных.
### Файл `typeController.js`:
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
### Файл `brandController.js`:
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
### Файл `TypeController.js`:
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

### Файл `brandController.js`:
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

## 8. Работа с файлами (загружаем картинки).
При создании device нам нужно будет подгружать картинку к каждому device.  
Библиотека для работы с картинками - `express-fileupload`.  
#### Зарегестрировать пакет в `index.js`:
```javascript
require("dotenv").config(); 
const express = require('express'); 
const sequelize = require("./db"); 
const cors = require("cors") 
const router = require("./routes/index"); 
const errorHandler = require("./middleware/ErrorHandingMiddleware"); 
const fileUpload = require("express-fileupload"); // импортируем fileupload

const PORT = process.env.PORT || 5000; 

conxt app = express(); 
app.use(cors()) 
app.use(express.json()) 
app.use("/api", router); 
app.use(fileUpload({})); // регестрируем fileUpload

app.use(errorHandler);
const start = async () => { 
  try {
    await sequelize.authenticate(); 
    await sequelize.sync() 
    app.listen(PORT, () => console.log(`Server started on port ${PORT}`)); 
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
### Файл `deviceController.js`:
```javascript
const uuid = require("uuid");
const { Device } = require("../models/models");
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
### Раздача СТАТИЧЕСКИХ файлов, которые находятся на нашем сервере.
Для раздачи статических файлов необходимо указать серверу явно, что файлы будут раздаваться из папки `static`.
### Файл `index.js`:
```javascript
require("dotenv").config(); 
const express = require('express'); 
const sequelize = require("./db"); 
const cors = require("cors") 
const router = require("./routes/index"); 
const errorHandler = require("./middleware/ErrorHandingMiddleware"); 
const fileUpload = require("express-fileupload"); 
const path = require("path") // импортируем библиотеку для универсального указания пути

const PORT = process.env.PORT || 5000; 

conxt app = express(); 
app.use(cors()) 
app.use(express.json()) 
app.use(express.static(path.resolve(__dirname, "static"))) // явно указываем из какой папки сервер будет раздовать статику
app.use(fileUpload({})); 
app.use("/api", router); 

app.use(errorHandler);
const start = async () => { 
  try {
    await sequelize.authenticate(); 
    await sequelize.sync() 
    app.listen(PORT, () => console.log(`Server started on port ${PORT}`)); 
  } catch (error) {
    console.log(error);
  }
};

start();
```

## 9. Получение устройств, фильтрация, пагинация - постраничный вывод.
### Файл `deviceController.js`:
```javascript
const uuid = require("uuid");
const path = require("path");
const { Device } = require("../models/models");
const ApiError = require("../error/ApiError");

class DeviceController {
  async create(req, res, next) {
    try {
      let { name, price, brandId, typeId, info } = req.body;
      const { img } = req.files;
      let fileName = uuid.v4() + ".jpg";
      img.mv(path.resolve(__dirname, "..", "static", fileName));

      const device = await Device.create({
        name,
        price,
        brandId,
        typeId,
        img: fileName,
      });

      return res.json(device);
    } catch (error) {
      next(ApiError.badRequest(error.message));
    }
  }

  async getAll(req, res) {
    // получаем brandId, typId из строки запроса req.query
    let { brandId, typeId, limit, page } = req.query;
    // переменные для отображения в ограниченном количестве
    page = page || 1; // страница
    limit = limit || 9; // количество device
    let offset = page * limit - limit; // отступ для отображения следующих device
    let devices; // переменная для хранения devices
    // Если brandId, typId не указаны, будем возвращать все divece
    if (!brandId && !typeId) {
      // из БД получаем все device
      devices = await Device.findAndCountAll({ limit, offset }); 
    }
    // фильтрация только по бренду
    if (brandId && !typeId) {
      // получаем все device в которых указан brand
      devices = await Device.findAndCountAll({
        where: { brandId },
        limit,
        offset,
      });
    }
    // фильтрация только по типу
    if (!brandId && typeId) {
      // получаем все device в которых указан type
      devices = await Device.findAndCountAll({
        where: { typeId },
        limit,
        offset,
      });
    }
    // фильтрация по типу и по бренду
    if (brandId && typeId) {
      // получаем все device в которых указан и type и brand
      devices = await Device.findAndCountAll({
        where: { typeId, brandId },
        limit,
        offset,
      });
    }
    return res.json(devices); // возвращаем все найденные device
  }
  async getOne(req, res) { }
}

module.exports = new DeviceController();
```
При использовании функции findAndCountAll в ответе от БД приходит поле count, в котором записано общее количесвто найденных позиций по фильтру brandId && typeId.

### info для device:
Добавляем в связь сущностей Device и DeviceInfo название поля которое будет у массива характеристик.
### Файл `models.js`:
```javascript
const sequelize = require("../db");
const { DataTypes } = require("sequelize");

const User = sequelize.define("user", {
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
  email: { type: DataTypes.STRING, unique: true },
  password: { type: DataTypes.STRING },
  role: { type: DataTypes.STRING, defaultValue: "USER" },
});

const Basket = sequelize.define("basket", {
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
});

const BasketDevice = sequelize.define("basket_device", {
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
});

const Device = sequelize.define("device", {
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
  name: { type: DataTypes.STRING, unique: true, allowNull: false },
  price: { type: DataTypes.FLOAT, allowNull: false },
  rating: { type: DataTypes.FLOAT, defaultValue: 0 },
  img: { type: DataTypes.STRING, allowNull: false },
});

const Type = sequelize.define("type", {
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
  name: { type: DataTypes.STRING, unique: true, allowNull: false },
});

const Brand = sequelize.define("brand", {
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
  name: { type: DataTypes.STRING, unique: true, allowNull: false },
});

const Rating = sequelize.define("rating", {
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
  rate: { type: DataTypes.INTEGER, allowNull: false },
});

const DeviceInfo = sequelize.define("device_info", {
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
  title: { type: DataTypes.STRING, allowNull: false },
  description: { type: DataTypes.STRING, allowNull: false },
});

const TypeBrand = sequelize.define("type_brand", {
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
});

User.hasOne(Basket);
Basket.belongsTo(User);

User.hasMany(Rating);
Rating.belongsTo(User);

Basket.hasMany(BasketDevice);
BasketDevice.belongsTo(Basket);

Type.hasMany(Device);
Device.belongsTo(Type);

Brand.hasMany(Device);
Device.belongsTo(Brand);

Device.hasMany(Rating);
Rating.belongsTo(Device);

Device.hasMany(BasketDevice);
BasketDevice.belongsTo(Device);
// добавляем название поля которое будет у массива характеристик {as: "info"}
Device.hasMany(DeviceInfo, {as: "info"});
DeviceInfo.belongsTo(Device);

Type.belongsToMany(Brand, { through: TypeBrand });
Brand.belongsToMany(Type, { through: TypeBrand });

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

### Файл `deviceController.js`:
```javascript
const uuid = require("uuid");
const path = require("path");
const { Device, DeviceInfo } = require("../models/models"); // добавляем DeviceInfo
const ApiError = require("../error/ApiError");

class DeviceController {
  async create(req, res, next) {
    try {
      let { name, price, brandId, typeId, info } = req.body;
      const { img } = req.files;
      let fileName = uuid.v4() + ".jpg";
      img.mv(path.resolve(__dirname, "..", "static", fileName));

      const device = await Device.create({
        name,
        price,
        brandId,
        typeId,
        img: fileName,
      });
      // если info есть в теле запроса
      if (info) {
        info = JSON.parse(info); // парсим данные, т.к. через form-data они приходят в формате строки
        info.forEach((i) => // пробегаемся по массиву
          DeviceInfo.create({ // создаем device с полем info
            title: i.title,
            description: i.description,
            deviceId: device.id,
          })
        );
      }

      return res.json(device);
    } catch (error) {
      next(ApiError.badRequest(error.message));
    }
  }
  async getAll(req, res) {
    let { brandId, typeId, limit, page } = req.query;
    page = page || 1;
    limit = limit || 9;
    let offset = page * limit - limit;
    let devices;
    if (!brandId && !typeId) {
      devices = await Device.findAndCountAll({ limit, offset });
    }
    if (brandId && !typeId) {
      devices = await Device.findAndCountAll({
        where: { brandId },
        limit,
        offset,
      });
    }
    if (!brandId && typeId) {
      devices = await Device.findAndCountAll({
        where: { typeId },
        limit,
        offset,
      });
    }
    if (brandId && typeId) {
      devices = await Device.findAndCountAll({
        where: { typeId, brandId },
        limit,
        offset,
      });
    }
    return res.json(devices);
  }
  async getOne(req, res) { }
}

module.exports = new DeviceController();
```
### Получение одного device (функция getOne()).
### Файл `deviceController.js`:
```javascript
const uuid = require("uuid");
const path = require("path");
const { Device, DeviceInfo } = require("../models/models");
const ApiError = require("../error/ApiError");

class DeviceController {
  async create(req, res, next) {
    try {
      let { name, price, brandId, typeId, info } = req.body;
      const { img } = req.files;
      let fileName = uuid.v4() + ".jpg";
      img.mv(path.resolve(__dirname, "..", "static", fileName));

      const device = await Device.create({
        name,
        price,
        brandId,
        typeId,
        img: fileName,
      });

      if (info) {
        info = JSON.parse(info);
        info.forEach((i) =>
          DeviceInfo.create({
            title: i.title,
            description: i.description,
            deviceId: device.id,
          })
        );
      }

      return res.json(device);
    } catch (error) {
      next(ApiError.badRequest(error.message));
    }
  }
  async getAll(req, res) {
    let { brandId, typeId, limit, page } = req.query;
    page = page || 1;
    limit = limit || 9;
    let offset = page * limit - limit;
    let devices;
    if (!brandId && !typeId) {
      devices = await Device.findAndCountAll({ limit, offset });
    }
    if (brandId && !typeId) {
      devices = await Device.findAndCountAll({
        where: { brandId },
        limit,
        offset,
      });
    }
    if (!brandId && typeId) {
      devices = await Device.findAndCountAll({
        where: { typeId },
        limit,
        offset,
      });
    }
    if (brandId && typeId) {
      devices = await Device.findAndCountAll({
        where: { typeId, brandId },
        limit,
        offset,
      });
    }
    return res.json(devices);
  }
  // Функция для получения одного devie
  async getOne(req, res) {
    const { id } = req.params; // получаем id из параметров которые в url строке
    const device = await Device.findOne({ // находим один device
      where: { id }, // device определенного id
      include: [{ model: DeviceInfo, as: "info" }], // include используется для подгрузки информации из другой сущности DeviceInfo с полем info
    });
    return res.json(device); // возвращаем device на клиент
  }
}

module.exports = new DeviceController();
```

## 10. Регистрация и авторизация. JWT токен. dcrypt.
### Файл `userController.js`:
#### Регистрация:
```javascript
require("dotenv").config();
const ApiError = require("../error/ApiError");
const bcrypt = require("bcrypt"); // импортируем bcrypt
const jwt = require("jsonwebtoken"); // импортируем jsonwebtoken
const { User, Basket } = require("../models/models"); // импортируем модели пользователя и корзины

const generateJwt = (id, email, role) => { // функция для генерации JWT токена, принимает id email кщду
  return jwt.sign(
    { id, email, role }, // первая часть JWT токена
    process.env.SECRET_KEY, // секретная фраза из файла .env
    { expiresIn: "24h",} // время активности токена
    );
};

class UserController {
  async registration(req, res, next) {
    const { email, password, role } = req.body; // получаем из тела мыло пароль и роль 
    if (!email || !password) { // если мыло или пароль пустые
      return next(ApiError.badRequest("Некорректный email или password")); // возвращаем ошибку
    }
    const candidate = await User.findOne({ where: { email } }); // находим пользователя с мылом в БД
    if (candidate) { // если такой пользователь уже есть
      return next(
        ApiError.badRequest("Пользователь с таким email уже существует!") // возвращаем ошибку
      );
    }
    // если мыло новое и пароль заполнен
    const hashPassword = await bcrypt.hash(password, 5); // хэшируем пароль
    const user = await User.create({ email, role, password: hashPassword }); // создаем нового пользователя
    const basket = await Basket.create({ userId: user.id }); // создаем корзину для пользователя
    const token = generateJwt(user.id, user.email, user.role); // генерируем JWT токен из id, email, role
    return res.json({ token }); // возвращаем 
  }
  async login(req, res, next) {}
  async check(req, res, next) {}
}

module.exports = new UserController();
```
### Файл `userController.js`:
#### Функция логина:
```javascript
require("dotenv").config();
const ApiError = require("../error/ApiError");
const bcrypt = require("bcrypt");
const jwt = require("jsonwebtoken");
const { User, Basket } = require("../models/models");

const generateJwt = (id, email, role) => {
  return jwt.sign({ id, email, role }, process.env.SECRET_KEY, {
    expiresIn: "24h",
  });
};

class UserController {
  async registration(req, res, next) {
    const { email, password, role } = req.body;
    if (!email || !password) {
      return next(ApiError.badRequest("Некорректный email или password"));
    }
    const candidate = await User.findOne({ where: { email } });
    if (candidate) {
      return next(
        ApiError.badRequest("Пользователь с таким email уже существует!")
      );
    }
    const hashPassword = await bcrypt.hash(password, 5);
    const user = await User.create({ email, role, password: hashPassword });
    const basket = await Basket.create({ userId: user.id });
    const token = generateJwt(user.id, user.email, user.role);
    return res.json({ token });
  }
  // Функция Login для пользователя
  async login(req, res, next) {
    const { email, password } = req.body; // получаем из тела ьыло и пароль
    const user = await User.findOne({ where: { email } }); // находим в БД такого пользователя
    if (!user) { // если такого пользователя нет
      return next(ApiError.internal("Пользователь с таким mail не найден")); // возвращаем ошибку
    }
    // проверяем пароль с помощью функции compareSync
    let comparePassword = bcrypt.compareSync(password, user.password);
    if (!comparePassword) { // если пароли не совпадают
      return next(ApiError.internal("Указан неверный пароль")); // возвращаем ошибку
    }
    // если логин и пароль верные
    const token = generateJwt(user.id, user.email, user.role); // генерируем JWT токен с помощью функции generateJwt
    return res.json({ token }); // возвращаем JWT токен на клиент
  }
  async check(req, res, next) {}
}

module.exports = new UserController();
```

## 11. middleware для Авторизации. Декодировать и проверять токен на валидность.
### Файл `authMiddleware.js` в папке middleware:
```javascript
const jwt = require("jsonwebtoken"); // импортируем jsonwebtoken

module.exports = function (req, res, next) {
  if (req.method === "OPTIONS") { // пропускаем метод OPTIONS
    next();
  }
  try {
    const token = req.headers.authorization.split(" ")[1]; // получаем токен из запроса, где req.headers.authorization.split(" ")[1] - это Тип токена
    if (!token) { // если токена нет
      return res.status(401).json({ message: "Не авторизован" }); // пользователь не авторизован
    }
    // раскодируем токен
    const decoded = jwt.verify(token, process.env.SECRET_KEY); // с помощью функции verify, куда передаем токен и секретную фразу
    req.user = decoded; // добавляем данные, которые мы вытащили к request
    next(); // вызываем следующий middleware
  } catch (error) { // если ошибка возникла
    res.status(401).json({ message: "Не авторизован" }); // возвращаем сообщение, о том что пользователь не авторизован
  }
};
```

### Файл `userController.js`:
```javascript
require("dotenv").config();
const ApiError = require("../error/ApiError");
const bcrypt = require("bcrypt");
const jwt = require("jsonwebtoken");
const { User, Basket } = require("../models/models");

const generateJwt = (id, email, role) => {
  return jwt.sign({ id, email, role }, process.env.SECRET_KEY, {
    expiresIn: "24h",
  });
};

class UserController {
  async registration(req, res, next) {
    const { email, password, role } = req.body;
    if (!email || !password) {
      return next(ApiError.badRequest("Некорректный email или password"));
    }
    const candidate = await User.findOne({ where: { email } });
    if (candidate) {
      return next(
        ApiError.badRequest("Пользователь с таким email уже существует!")
      );
    }
    const hashPassword = await bcrypt.hash(password, 5);
    const user = await User.create({ email, role, password: hashPassword });
    const basket = await Basket.create({ userId: user.id });
    const token = generateJwt(user.id, user.email, user.role);
    return res.json({ token });
  }
  async login(req, res, next) {
    const { email, password } = req.body;
    const user = await User.findOne({ where: { email } });
    if (!user) {
      return next(ApiError.internal("Пользователь с таким mail не найден"));
    }
    let comparePassword = bcrypt.compareSync(password, user.password);
    if (!comparePassword) {
      return next(ApiError.internal("Указан неверный пароль"));
    }
    const token = generateJwt(user.id, user.email, user.role);
    return res.json({ token });
  }
  // добавляем функцию check
  async check(req, res, next) {
    const token = generateJwt(req.user.id, req.user.email, req.user.role); // генерация нового токена. Если пользователь постоянно использует свой аккаунт токен у него будет перезаписываться.
    return res.json({ token }); // отправка нового токена клиенту
  }
}

module.exports = new UserController();
```
### Файл `userRouter.js`:
```javascript
const Router = require("express");
const router = new Router();
const userController = require("../controllers/userController");
const authMiddleware = require("../middleware/authMiddleware"); // импортируем полученный middleware для проверки пользователя

router.post("/registration", userController.registration);
router.post("/login", userController.login);
router.get("/auth", authMiddleware, userController.check); // добавляем проверку при вызове get запроса 

module.exports = router;
```

## 12. middleware для Проверки Роли Пользователя. 
Добавлять типы, бренды, товары может только АДМИНИСТРАТОР.
### Файл `chekRoleMiddleware.js` в папке middleware:
```javascript
const jwt = require("jsonwebtoken");

module.exports = function (role) { // экспортируем функцию, которая принимает параметром role
  return function (req, res, next) { // возвращаем функцию
    if (req.method === "OPTIONS") { // пропускаем метод OPTIONS
      next(); // сразу вызываем следующую функцию
    }
    try {
      const token = req.headers.authorization.split(" ")[1]; // получаем токен
      if (!token) { // если токена нет
        return res.status(401).json({ message: "Не авторизован" }); // выводим сообщение
      }
      // декодируем полученный токен
      const decoded = jwt.verify(token, process.env.SECRET_KEY);
      if (decoded.role !== role) { // если роли не совпадают
        return res.status(403).json({ message: "Нет доступа" }); // выдаем сообщение о том, что нет доступа
      }
      // если роль нужная
      req.user = decoded; // помещаем информацию в request 
      next(); // вызываем следующий middleware
    } catch (error) {
      res.status(401).json({ message: "Не авторизован" }); // сообщение об ошибки
    }
  };
};
```
### Файл `typeRouter.js`:
```javascript
const Router = require("express");
const router = new Router();
const typeController = require("../controllers/typeController");
const checkRole = require("../middleware/checkRoleMiddleware"); // импортируем функцию проверки роли

router.post("/", checkRole("ADMIN"), typeController.create); // проверяем роль перед созданием ТИПА
router.get("/", typeController.getAll);

module.exports = router;
```
### Файл `brandRouter.js`.
```javascript
const Router = require("express");
const router = new Router();
const brandController = require("../controllers/brandController");
const checkRole = require("../middleware/checkRoleMiddleware"); // импортируем функцию проверки роли

router.post('/', checkRole("ADMIN"), brandController.create)
router.get('/', brandController.getAll)

module.exports = router;
```

## Дополнительная информация.
## 1. Что такое request и response при работе с сайтами?
`Request` (запрос) и `response` (ответ) - это основные компоненты взаимодействия клиента и сервера в веб-приложениях.
- `Request` - это **сообщение, отправленное клиентом** (например, браузером) на сервер. Оно содержит информацию о том, что клиент хочет получить или отправить. Запрос может включать метод (GET, POST, PUT, DELETE и т.д.), URL, заголовки и тело запроса.
- `Response` - это **сообщение, которое сервер отправляет обратно клиенту** в ответ на запрос. Оно содержит статус выполнения запроса (например 200 ОК, 400 Not Found), заголовки и, возможно, тело ответа с запрашиваемыми данными.

### Методы request и response в фреймворке Express.js
#### 1. Методы объекта `request` (req):

- `req.params`: Получает параметры из URl
```javascript
app.get('/users/:id', (req, res) => {
  const userId = req.params.id; // Получаем ID пользователя из параметров
  res.send(`User ID: ${userId}`);
});

```
- `req.query`: Получает параметры из строки запроса
```javascript
app.get('/search', (req, res) => {
    const searchTerm = req.query.q; // Получаем параметр q из строки запроса
    res.send(`Search term: ${searchTerm}`);
});
```
- `req.body`: Получает параметры из тела запроса (требует middleware типа body-parser)
```javascript
app.post('/users', (req, res) => {
    const newUser = req.body; // Получаем данные из тела запроса
    res.status(201).json(newUser); // Отправляем созданного пользователя
});
```
- `req.method`: Получает HTTP-метод запроса
```javascript
app.use((req, res, next) => {
    console.log(`HTTP Method: ${req.method}`); // Логируем метод запроса
    next();
});
```

#### 2. Методы объекта `response` (res):
- `res.send()`: Отправляет ответ клиенту.
```javascript
app.get('/', (req, res) => {
    res.send('Hello World!'); // Отправляем текстовый ответ
});
```
- `res.json()`: Отправляет ответ в формате JSON.
```javascript
app.get('/data', (req, res) => {
    const data = { name: 'John', age: 30 };
    res.json(data); // Отправляем объект в формате JSON
});
```
- `res.status()`: Устанавливает статус ответа.
```javascript
app.get('/not-found', (req, res) => {
    res.status(404).send('Page not found'); // Устанавливаем статус 404 и отправляем сообщение
});
```
- `res.redirect()`: Перенаправляет на другой URL.
```javascript
app.get('/old-route', (req, res) => {
    res.redirect('/new-route'); // Перенаправляем на новый маршрут
});
```
- `res.render()`: Отправляет HTML-страницу, используя шаблонизатор.
```javascript
app.get('/profile', (req, res) => {
    res.render('profile', { user: req.user }); // Рендерим страницу профиля с данными пользователя
});
```

### Примеры работы с request и response в методах Express
1. `GET`: **Получение данных**.
```javascript
app.get('/users', (req, res) => {
    // Получаем список пользователей
    res.json(users);
});
```
2. `POST`: **Создание новой записи**.
```javascript
app.post('/users', (req, res) => {
    const newUser = req.body; // Получаем данные из тела запроса
    users.push(newUser); // Добавляем пользователя в массив
    res.status(201).json(newUser); // Отправляем созданного пользователя
});
```
3. `PUT`: **Обновление существующей записи**.
```javascript
app.put('/users/:id', (req, res) => {
    const userId = req.params.id; // Получаем ID пользователя из параметров
    const updatedUser = req.body; // Получаем обновленные данные
    // Логика обновления пользователя
    res.json(updatedUser); // Отправляем обновленного пользователя
});
```
4. `DELETE`: **Удаление записи**.
```javascript
app.delete('/users/:id', (req, res) => {
    const userId = req.params.id; // Получаем ID пользователя из параметров
    // Логика удаления пользователя
    res.status(204).send(); // Отправляем статус 204 No Content
});
```

## 2. Что такое REST API?
**REST API** (Representational State Transfet Application Programming Interface) - это стандарт архитектуры программного обеспечения, который определяет **правила** и **ограничения** для создания веб-сервисов. **REST API** использует **HTTP** для **обмена данными** между клиентом и сервером. Он основан на принципах **REST**, котороые описывают, как взаимодействовать с ресурсами через стандартные **операции HTTP,** такие как **GET**, **POST**, **PUT**, **DELETE**.  
**REST API** позволяет клиентам **получать**, **создавать**, **обновлять** и **удалять** данные на сервере. Он использует уникальные **URL-адреса** для идентификации ресурсов и возвращает данные в формате **JSON** или **XML**.

### Использование REST API для интернет-магазина.
1. **Получение информации о продуктах**. Вы можете использовать REST API для получения информации о продуктах, таких как название, описание, цена и наличие. Это позволяет отображать актуальную информацию о продуктах на вашем веб-сайте или в мобильном приложении.

2. **Добавление продуктов в корзину**. REST API может быть использован для добавления выбраных продуктов в корзину покупателя. После добавления продуктов в корзину, вы можете сохранить информацию о корзине на сервере и обновлять ее при необходимости.

3. **Оформление заказа**. REST API позволяет оформить заказ, передавая необходимую информацию, такую как адрес доставки, способ оплаты и выбранные продукты. Сервер может обработать эту информацию и создать заказ в системе интернет-магазина.

4. **Получение информации о заказах**. REST API может быть использован для получения информации о заказах, таких как статус, дата и время доставки, и детали покупателя. Это позволит вам отслеживать и управлять заказами в вашем интернет-магазине.

5. **Аутентификация и авторизация**. REST API может быть использован для аутентификации пользователей и предоставления доступа к определенным функциям или ресурсам. Вы можете использовать токены аутентификации, такие как JWT, для обеспеччения безопасности и контроля доступа к данным.

## 3. Что такое ORM для реляционных баз данных?
**ORM (Object-Relational Mapping)** - это технология, которая позволяет **работать с реляционными базами данных**, используя объектно-ориентированный подход. Она представляет средства для **отображения данных** из таблиц базы данных на объекты в программном коде и обратно.  
**ORM упрощает взаимодействие с базой данных**, позволяя разработчикам использовать объекты и методы для выполнения операций **CRUD (создание, чтение, обновление, удаление)** **вместо написания прямых SQL-запросов**. Он так же обеспечивает **автоматическую генерация SQL-запросов на основе объектов и их отношений**.

ORM может быть полезным инструментом при разработке приложений, особенно в случае сложных структур баз данных или при работе с большим объемом данных.

Примеры ORM:
- Hibernate (для Java)
- SQLAlchemy (для Python)
- Entity Framework (для .NET)
- Django ORM (для Python и Django фреймворка)
- ActiveRecord (для Ruby on Rails)
- Sequelize (для Node.js)

## 4. Что такое JWT авторизация и JWT токен?
**JWT (JSON Web Token)** - это **стандарт** для создания **токенов**, которые пользуются для **аутентификации** и **авторизации** в веб-приложениях.

Токены JWT состоят из трех частей:
- **заголовка** (header)
- **полезной нагрузки** (payload)
- **подписи** (signature)

`header` содержит информацию о типе токена и используемом алгоритме шифрования или подписи.

`payload` содержит данные, которые могут быть использованы для идентификации пользователя или передачи другой информации. В полезной нагрузке могут быть указаны такие данные, как идентификатор пользователя, срок действия токена и другие пользовательские данные.

`signature` используется для проверки целостности токена и подтвержения его подлинности. Подпись создается с использованием секретного ключа, который известен только серверу, генерирующему токен.

**JWT токены широко используются для аутентификации и авторизации в веб-приложениях.** Они компактны, безопасны для передачи через сеть и могут быть использованы для передачи информации между клиентом и сервером.

### Применение JWT токенов
JWT токены могут быть использованы для различных целей:
- **Аутентификация**. После успешной аутотентификации пользователя, сервер может выдать JWT токен, который будет содержать информацию об аутентифицированном пользователе. Этот токен может быть передан клиенту и использован для последующей авторизации запросов к защищенным ресурсам.

- **Авторизация**. JWT токены могут содержать информацию о разрешениях пользователя, которая может быть использована для определения доступа к определенным ресурсам или функциональности.

- **Одноэтаптаня аутентификация**. JWT токены могут быть использованы для реализации одноэтапной аутентификации (Single Sing-On, SSO), позволяющей пользователям получить доступ к нескольким приложениям или сервисам без необходимости повторной аутентификации.

### Преимущества JWT токенов.
- **Без состояния**. JWT токены не требуют хранения состояния на сервере, что делает их масштабируемыми и удобными для использования в распределенных системах.

- **Компактность**. JWT токены имеют компактный формат, что делает их удобными для передачи через сеть.

- **Расширяемость**. JWT токены могут содержать пользовательские данные в полезной нагрузке, что позволяет передать дополнительную информацию, необходимую для приложений.

### Пример JWT токена который состоит из трех частей, разделенных точками: заголовка, полезной нагрузки и подписи. Каждая часть закодирована в формате Base64.
**JWT-токен**:
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxLCJleHAiOjE1ODEzNTcwMzl9.E4FNMef6tkjIsf7paNrWZnB88c3WyIfjONzAeEd4wF0
```
**header**:
```javascript
{
  "alg": "HS256",
  "typ": "JWT"
}
```
**payload**:
```javascript
{
  "user_id": 1,
  "exp": 1581357039
}
```
**signature**:
```javascript
E4FNMef6tkjIsf7paNrWZnB88c3WyIfjONzAeEd4wF0
```

## 5. Что такое Bearer в токене?
`Bearer` — это тип схемы аутентификации, используемой в заголовке `Authorization` **HTTP-запроса**. Когда вы видите токен, начинающийся с `Bearer`, это означает, что токен является `"носителем"` (bearer token), который предоставляет доступ к защищенным ресурсам на сервере.

### Как работает Bearer Token?
1. **Аутентификация**: Когда пользователь **успешно аутентифицируется** (например, вводит свои учетные данные), **сервер генерирует токен** (например, JWT) и **отправляет его обратно клиенту**.
2. **Использование токена**: **Клиент хранит этот токен** и **включает его в заголовок Authorization при каждом запросе к защищенным ресурсам**.  
**Например**:
```
Authorization: Bearer eyJhbGciOiJIUzIasxas1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvasxZSBEb2UiLCJhZG1pbiI6dHJ1ZSwiaWF0IjoxNTE2MjM5MDIyfQ.SdUMMAsFFk83SHyqGjfxsaasxQ1pt8v_8t4WZku5Ty3CLMaH4
```
3. **Проверка токена**: **Сервер проверяет токен**, чтобы убедиться, что он **действителен и не истек**. **Если токен действителен, сервер предоставляет доступ к запрашиваемому ресурсу**.

### Типы токенов
1. `Bearer Tokens`: Это тип токенов, который используется для **аутентификации и авторизации**. `Bearer` токены передаются в заголовке `Authorization` HTTP-запроса с префиксом `Bearer`. Они предоставляют доступ к защищенным ресурсам на сервере и упрощают процесс аутентификации. После успешной аутентификации сервер **генерирует** `Bearer` токен (например, `JWT`) и **отправляет** его клиенту, который затем **включает его в заголовок при каждом запросе**.
2. `Opaque Tokens`: Это токены, которые **не содержат информации о пользователе** и **не могут быть декодированы без обращения к серверу**. Сервер хранит информацию о токене и проверяет его при каждом запросе.
3. `JWT (JSON Web Token)`: Хотя `JWT` часто используется как **bearer токен**, он сам по себе является **форматом токена, который может содержать информацию о пользователе и сроке действия**.
4. `Refresh Tokens`: Эти токены используются для **получения новых access-токенов** после истечения срока действия. Они обычно имеют более длительный срок действия и могут быть использованы для обновления `JWT`.
5. `SAML Tokens`: Используются в контексте `SSO (Single Sign-On)` и представляют собой `XML-документы`, которые содержат информацию о пользователе и его аутентификации.
6. `API Tokens`: Это токены, которые используются **для аутентификации при доступе к API**. Они могут быть как `JWT`, так и **другими типами токенов**.
7. `Session Tokens`: Эти токены используются для **управления сессиями пользователей**. Они **могут хранить информацию о состоянии сессии и аутентификации**.

## 6. Что такое CORS?
### Проверка обработки запросов
**CORS (Cross-Origin Resourse Sharing)** обеспечивает **проверку обработки запросов от других доменов** с помощью механизма, который включает в себя **заголовки HTTP** и **предварительные запросы** (preflight requets)

1. `Заголовок Origin`:
Когда браузер отправляет запрос к ресурсу на другом домене, он добавляет заголовок Origin, который указывает источник (домен) запроса. Сервер, получив этот запрос, может проверить, разрешен ли доступ с этого источника.

2. `Ответ сервера`:
Если сервер поддерживает CORS, он должен вернуть соответствующие заголовки в ответе. Например, заголовок Access-Control-Alow-Origin указывает, какие домены могут получить доступ к ресурсу. Если домен, указанный в Origin, совпадает с разрешенными, браузер разрешает выполнение запроса.

3. `Предварительные запросы` (Preflight requets):
Для некоторых типов запросов (например, с методами PUT, DELETE или с нестандартными заголовками) браузер сначала отправит предварительный запростипа OPTIONS. Этот запрос проверяет, поддерживает ли сервер CORS и какие методы в заголовке разрешены. Сервер должен ответить с заголовками, которые указывают, что запрос должен быть выполнен.

### Как ограничить пул разрешенных доменов?
```javascript
const express = require("express");
const cors = require("cors");

const app = express();

// Настройка CORS с ограничением по доменам
const allowedDomains = ['http://example.com', 'http://anotherdomain.com'];

const corsOptions = {
    origin: function (origin, callback) {
        // Разрешаем запросы без указания источника (например, для локального тестирования)
        if (!origin || allowedDomains.indexOf(origin) !== -1) {
            callback(null, true);
        } else {
            callback(new Error('Not allowed by CORS'));
        }
    }
};

// Подключение CORS с настройками
app.use(cors(corsOptions));

// Пример маршрута
app.get("/api/data", (req, res) => {
    res.json({ message: "Hello from the server!" });
});

// Запуск сервера
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
```
### Объяснение кода:
1. `allowedDomains` - Массив, содержащий домены, которым разрешен доступ к вашему API
2. `corsOptions` - Объект с настройками CORS, в котором функция origin проверяет, находится ли запрашиваемый домен в списке разрешенных. Если домен разрешен, запрос проходит, иначе возвращается ошибка.
3. `app.use(cors(corsOptions))` - Подключает CORS с указаными настройками.

### Дефолтные настройки CORS для интернет-магазина.
1. **Разрешенные источники**:
Обычно стоит разрешить доступ только с вашего основного домена (например, `https://www.yourstore.com`). Это предотвратит доступ к вашему API с других, потенциально небезопасных доменов.

2. **Методы**:
Разрешите только необходимые методы HTTP. Для интернет-магазина это могут быть GET, POST, PUT, и DELETE. Например:
```javascript
const corsOptions = {
    origin: 'https://www.yourstore.com',
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
};
```

3. **Заголовки**:
Убедитесь, что вы разрешаете только те заголовки, которые необходимы для вашего приложения. Например, если вы используете авторизацию, вам может понадобиться разрешить заголовок Authorization:
```javascript
const corsOptions = {
    allowedHeaders: ['Content-Type', 'Authorization'],
};
```

4. **Предварительные запросы**:
Если ваше приложение использует сложные запросы (например, с нестандартными заголовками), убедитесь, что сервер правильно обрабатывает предварительные запросы (preflight requests) с методом OPTIONS.

5. **Безопасность**:
Рассмотрите возможность использования HTTPS для вашего интернет-магазина, чтобы защитить данные пользователей и транзакции.

### Пример настройки CORS в Express может выглядеть так:
```javascript
const express = require("express");
const cors = require("cors");

const app = express();

const corsOptions = {
    origin: 'https://www.yourstore.com',
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    allowedHeaders: ['Content-Type', 'Authorization'],
};

app.use(cors(corsOptions));

// Пример маршрута
app.get("/api/products", (req, res) => {
    res.json({ message: "Список продуктов" });
});

// Запуск сервера
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Сервер запущен на порту ${PORT}`);
});
```

## 7. Что такое ВАЛИДАЦИЯ?
**Валидация данных** - это процесс **проверки корректности и целостности данных**, которые вводятся или обрабатываются в приложении. Основная цель валидации - убедиться, что данные **соответствуют определенным критериям и требованиям**, прежде чем они будут использованы в системе.

### Зачем нужна валидация данных?
1. `Предотвращение ошибок`. Валидация помогает избежать ошибок, которые могут возникнуть из-за некорректных или неполных данных. Например, если пользователь введет неверный формат электронной почты. 

2. `Безопасность`. Валидация данных так же важна для защиты приложения от атак, таких как SQL-инъекции или XSS (межсайтовые скрипты). Проверка входящих данных помогает предотвратить выполнение вредоносного кода.

3. `Улучшение пользовательского опыта`. Когда данные валидируются на этапе ввода, пользователь получает мнгновенную обратноую связь. 

### Примеры валидации данных.
- Проверка, что поле "электронный адрес" содержит корректный адрес.
- Убедиться, что пароль соответствуют определенным требованиям (например, длина от 8 до 20 символов, наличие цифр и спецсимволов)
- Проверка, что дата рождения не указывает на будущее.

Валидация данных может выполняться как на стороне клиента (с помощью JavaScript), так и на стороне сервера (использование библиотек для валидации на Node.js)

### Библиотеки для валидации на Node.js
1. **Joi**
2. **Yup**
3. **express-validator**
4. **Validator.js**
5. **Ajv**

## 8. Использование super() при вызове конструктора в унаследованном классе.
```javascript
class ApiError extends Error {
  constructor(status, message) {
    super();
    this.status = status;
    this.message = message;
  }
  static badRequest(message) {
    return new ApiError(404, message);
  }
  static internal(message) {
    return new ApiError(500, message);
  }
  static forbidden(message) {
    return new ApiError(403, message);
  }
}

module.exports = ApiError;
```

### При вызове super() происходит:
1. **Инициализация родительского класса**: Вызов `super()` позволяет инициализировать свойства, определенные в родительском классе Error. Это важно, чтобы экземпляр ApiError имел все необходимые свойства, такие как message и stack, которые предоставляет класс Error.

2. **Передача аргументов**: Если вы хотите передать аргументы в родительский класс, вы можете сделать это через super(message), где message — это сообщение об ошибке. В вашем коде это можно было бы сделать так:
```javascript
super(message);
```
Это обеспечит правильную инициализацию сообщения об ошибке в объекте ApiError.

3. **Поддержка прототипного наследования**: Вызов `super()` также устанавливает правильную цепочку прототипов, что позволяет экземплярам ApiError корректно восприниматься как экземпляры Error. Это важно для обработки ошибок, так как позволяет использовать стандартные механизмы обработки ошибок JavaScript, такие как try...catch.

Таким образом, вызов `super()` в конструкторе класса ApiError обеспечивает правильную инициализацию и функциональность, унаследованную от класса Error.

## 9. Проектирование и написание middleware в Node.js
**Middleware** в Node.js, особенно в рамках фреимворка `Express`, представляют собой функции, которые **обрабатывают запросы и ответы**. Они могут выполнять различные задачи, такие как логирование, аутентификация, обработка ошибок и многое другое.

### Основные принципи middleware:
1. **Структура middleware**:
Каждый middleware - это функция **которая принмает три аргумента**: `request`, `response`, `next`.
- `request` - объект запроса.
- `response` - объект ответа.
- `next` - функция, которая передает управление следующему **middleware** в цепочке.

2. **Вызов next()**:
Важно вызвать `next()` в конце выполнения **middleware**, что бы передать управдение **следующему middleware**. Если `next()` не будет вызван, запрос "зависнет", и последующие middleware не будут выполнены.

### Список и реализация middleware для онлайн-магазина.
В онлайн-магазине с функционалом **регистрации**, **логина**, **добавления товара** в корзину и **покупки** можно использовать несколько **middleware** для обработки запросов. Вот примерный список middleware и их реализация:
1. `Логирование запросов`
2. `Аутентификация пользователя`
3. `Валидация данных`
4. `Обработка ошибок`
5. `Управление сессиями`

### Реализация middleware
```javascript
const express = require('express');
const session = require('express-session');
const bodyParser = require('body-parser');

const app = express();
app.use(bodyParser.json());
app.use(session({ secret: 'секрет', resave: false, saveUninitialized: true }));

// 1. Middleware для логирования запросов
app.use((req, res, next) => {
  console.log(`Запрос к ${req.url} с методом ${req.method}`);
  next();
});

// 2. Middleware для аутентификации пользователя
const authenticate = (req, res, next) => {
  if (req.session.user) {
    next(); // Пользователь аутентифицирован, продолжаем
  } else {
    res.status(401).send('Необходима аутентификация'); // Неаутентифицированный доступ
  }
};

// 3. Middleware для валидации данных
const validateProduct = (req, res, next) => {
  const { productId, quantity } = req.body;
  if (!productId || !quantity || quantity <= 0) {
    return res.status(400).send('Неверные данные продукта');
  }
  next(); // Данные валидны, продолжаем
};

// 4. Middleware для обработки ошибок
const errorHandler = (err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Что-то пошло не так!');
};

// Пример маршрутов
app.post('/register', (req, res) => {
  // Логика регистрации пользователя
  res.send('Пользователь зарегистрирован');
});

app.post('/login', (req, res) => {
  // Логика логина пользователя
  req.session.user = req.body.username; // Сохраняем пользователя в сессии
  res.send('Пользователь вошел в систему');
});

app.post('/cart/add', authenticate, validateProduct, (req, res) => {
  // Логика добавления товара в корзину
  res.send('Товар добавлен в корзину');
});

app.use(errorHandler); // Обработка ошибок должна быть в конце

// Запуск сервера
app.listen(3000, () => {
  console.log('Сервер запущен на http://localhost:3000');
});
```
### Объяснение:
1. `Логирование запросов`. Этот middleware выводит информацию о каждом запросе.
2. `Аутентификация пользователя`. Проверяет, есть ли у пользователя активная сессия. Если нет, возвращает статус 401.
3. `Валидация данных`. Проверяет, что данные, отправленные для добавления в корзину, корректны. Если данные не верные, возвращает статус 400.
4. `Обработка ошибок`. Этот middleware обрабатывает любые ошибки, возникающие в приложении, и возвращает статус 500.
5. `Маршруты`. Определеные маршруты для регистрации, логирования и добавления товара в корзину. Для добавления товара в корзину используется аутентификация и валидация данных.

## 10. Методы объектов Sequelize
1. `sync()`: Синхронизирует модель с базой данных, создавая таблицы, если они не существуют.
```javascript
await sequelize.sync();
```
2. `authenticate()`: Проверяет соединение с базой данных.
```javascript
await sequelize.authenticate();
```
3. `close()`: Закрывает соединение с базой данных.
```javascript
await sequelize.close();
```
4. `transaction()`: Создает новую транзакцию для выполнения нескольких операций как единое целое.
```javascript
const t = await sequelize.transaction();
try {
    // операции с использованием транзакции
    await t.commit();
} catch (error) {
    await t.rollback();
}
```
5. `define()`: Определяет новую модель.
```javascript
const User = sequelize.define('user', {
    name: DataTypes.STRING,
    age: DataTypes.INTEGER
});
```
6. `query()`: Выполняет произвольный SQL-запрос.
```javascript
const result = await sequelize.query("SELECT * FROM users");
```
7. `model()`: Получает модель по имени.
```javascript
const User = sequelize.model('user');
```
8. `addHook()`: Добавляет хуки (обработчики событий) к модели.
```javascript
User.addHook('beforeCreate', (user, options) => {
    // логика перед созданием пользователя
});
```
### Схема работы методов Sequelize
1. **Определение модели**: Сначала вы определяете модель с помощью `define()`, указывая структуру таблицы.
2. **Синхронизация**: Метод `sync()` создает таблицы в базе данных на основе определенных моделей.
3. **Взаимодействие с базой данных**: Вы можете использовать методы для выполнения операций `CRUD` (создание, чтение, обновление, удаление) и другие действия, такие как транзакции и выполнение произвольных запросов.
4. **Закрытие соединения**: После завершения работы с базой данных вы можете закрыть соединение с помощью `close()`.

### Встроенные методы объектов типа Sequelize
1. `findAll`: Получает все записи из таблицы.
```javascript
const types = await Type.findAll();
```
2. `findOne`: Получает одну запись по заданным условиям.
```javascript
const type = await Type.findOne({ where: { id: 1 } });
```
3. `update`: Обновляет существующие записи.
```javascript
await Type.update({ name: 'Новое имя' }, { where: { id: 1 } });
```
4. `destroy`: Удаляет записи из таблицы.
```javascript
await Type.destroy({ where: { id: 1 } });
```
5. `count`: Подсчитывает количество записей в таблице.
```javascript
const count = await Type.count();
```
6. `bulkCreate`: Создает несколько записей за один запрос.
```javascript
await Type.bulkCreate([{ name: 'Тип 1' }, { name: 'Тип 2' }]);
```
### Схема работы кода
- `Определение модели`: Вы создаете модель с помощью sequelize.define, где указываете структуру таблицы.
- `Создание записи`: Вызываете метод create, который формирует SQL-запрос для вставки данных в базу.
- `Взаимодействие с базой данных`: Sequelize обрабатывает взаимодействие с базой данных, включая создание, чтение, обновление и удаление записей.

### Основные методы и их параметры.
**Пример записи функции с получением данных из БД**:
```javascript
async getOne(req, res) {
  const { id } = req.params;
  const device = await Device.findOne({
    
    where: { id }, // условия для фильтрации
    include: [{ model: DeviceInfo, as: "info" }], // включить другую модель (сущность)

  });
  return res.json(device);
}
```

### 1. `findAll` - Используется для **получения всех записей из таблицы**.  
#### Часто используемые параметры:
- `where`: **Условия для фильтрации записей**. Например:
```javascript
where: { status: 'active' }
```
- `attributes`: **Указывает, какие поля выбрать**. Например:
```javascript
attributes: ['id', 'name']
```
- `include`: **Позволяет включать связанные модели**. Например:
```javascript
include: [{ model: DeviceInfo, as: 'info' }]
```
- `order`: **Указывает порядок сортировки**. Например:
```javascript
order: [['createdAt', 'DESC']]
```
- `limit`: **Ограничивает количество возвращаемых записей**. Например:
```javascript
limit: 10
```
- `offset`: **Пропускает указанное количество записей**. Например:
```javascript
offset: 5
```
### 2. `findOne` - Используется для **получения одной записи из таблицы**.
#### Часто используемые параметры:
- `where`: **Условия для поиска конкретной записи**. Например:
```javascript
where: { id: 1 }
```
- `attributes`: **Указывает, какие поля выбрать**. Например:
```javascript
attributes: ['id', 'name']
```
- `include`: **Включает связанные модели**. Например:
```javascript
include: [{ model: DeviceInfo, as: 'info' }]
```
### 3. `findCount` - **Используется для подсчета количества записей, соответствующих условиям**.
#### Часто используемые параметры:
- `where`: **Условия для фильтрации записей**. Например:
```javascript
where: { status: 'active' }
```
### 4. `findOrCreate` - **Находит запись или создает новую, если она не существует**.
#### Часто используемые параметры:
- `where`: **Условия для поиска записи**. Например:
```javascript
where: { email: 'example@example.com' }
```
- `defaults`: **Значения по умолчанию для новой записи, если она создается**. Например:
```javascript
defaults: { name: 'New User' }
```
### 5. `destroy` - **Удаляет записи из таблицы**.
#### Часто используемые параметры:
- `where`: **Условия для удаления записей**. Например:
```javascript
where: { id: 1 }
```
### 6. `update` - Обновляет записи в таблице.
#### Часто используемые параметры:
- `where`: **Условия для обновления записей**. Например:
```javascript
where: { id: 1 }
```
- `fields`: **Указывает, какие поля обновить**. Например:
```javascript
fields: ['name', 'status']
```

## 11. Метод OPTIONS в Express.
### Декодирование токена и проверка его на валидность:
```javascript
const jwt = require("jsonwebtoken"); // импортируем jsonwebtoken

module.exports = function (req, res, next) {
  if (req.method === "OPTIONS") {
    next();
  }
  try {
    const token = req.headers.authorization.split(" ")[1];
    if (!token) {
      return res.status(401).json({ message: "Не авторизован" });
    }
    const decoded = jwt.verify(token, process.env.SECRET_KEY);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ message: "Не авторизован" });
  }
};
```
Метод `OPTIONS` в **HTTP** используется для **запроса информации о поддерживаемых методах и параметрах**, которые сервер может **обрабатывать для определенного ресурса**.  
Это полезно для **проверки**, какие методы доступны для **конкретного URL**, и часто используется в контексте **кросс-доменных запросов** (CORS).

В вашем коде, проверка на `req.method === "OPTIONS"` позволяет серверу **обрабатывать запросы типа OPTIONS, не выполняя дальнейшую логику аутентификации**.  
Это **важно**, потому что **браузеры могут отправлять запросы OPTIONS перед основными запросами (например, POST или GET) для проверки**, разрешены ли такие запросы.
### Другие HTTP методы
- `GET`: Используется для получения данных с сервера.
- `POST`: Используется для отправки данных на сервер, например, для создания нового ресурса.
- `PUT`: Используется для обновления существующего ресурса.
- `DELETE`: Используется для удаления ресурса.
- `PATCH`: Используется для частичного обновления ресурса.
- `HEAD`: Похож на GET, но возвращает только заголовки, без тела ответа.
- `CONNECT`: Используется для установки туннеля к серверу, обычно для работы с прокси-серверами.
- `TRACE`: Позволяет клиенту получить диагностическую информацию о пути, по которому проходит запрос к серверу.

### Что могут содержать методы?
Каждый метод может содержать различные **заголовки** и **тело запроса**:
- `GET`: Обычно не содержит тела, но может содержать параметры в URL.
- `POST`: Содержит данные в теле запроса, которые могут быть в формате JSON, формата URL-кодирования и т.д.
- `PUT/PATCH`: Также содержат данные в теле запроса, которые представляют собой обновления для ресурса.
- `DELETE`: Обычно не содержит тела, но может содержать заголовки для аутентификации.

### Для чего проверять методы?
1. Проверка методов важна для:
2. **Безопасности**: Некоторые методы могут быть опасными, например, DELETE или PUT, и их следует обрабатывать с осторожностью.
3. **Управления доступом**: Вы можете ограничить доступ к определенным методам для разных пользователей или ролей.
4. **Оптимизации**: Зная, какие методы поддерживаются, клиент может оптимизировать свои запросы и избежать ненужных операций.

### Почему мы пропускаем проверку для OPTIONS?
Если метод запроса равен `OPTIONS`, вы вызываете `next()`, что позволяет перейти к следующему обработчику **без выполнения дальнейшей логики аутентификации**. Это делается по нескольким причинам:
1. **Кросс-доменные запросы (CORS)**: Браузеры часто отправляют запросы `OPTIONS` **перед основными запросами** (например, `POST` или `GET`), чтобы проверить, **разрешены ли такие запросы**. Если сервер не обрабатывает `OPTIONS`, это может привести **к ошибкам при выполнении кросс-доменных запросов**.
2. **Отсутствие необходимости в аутентификации**: Запросы `OPTIONS` обычно **не требуют аутентификации**, так как они **предназначены для получения информации о доступных методах**. Пропуская проверку, вы позволяете **клиентам узнать**, **какие методы доступны**, без необходимости предоставлять токен или другую информацию для аутентификации.
3. **Упрощение логики**: Обработка `OPTIONS` отдельно позволяет **избежать ненужных проверок и упрощает логику вашего приложения**. Это делает код более чистым и понятным.