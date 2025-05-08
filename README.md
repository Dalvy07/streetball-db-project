# База данных для приложения Streetball-App

![Логотип](https://api.placeholder.com/400/150?text=StreetBall+App)

## Проект базы данных NoSQL MongoDB

**Автор:** Владислав Люлька  

---

Люблин, 2025

# Документация проекта базы данных StreetBall App

## Содержание
1. [Описание проекта](#1-описание-проекта)
2. [Функциональные требования к базе данных](#2-функциональные-требования-к-базе-данных)
3. [Структура коллекций](#3-структура-коллекций)
4. [Реализация базы данных](#4-реализация-базы-данных)
5. [Запросы к базе данных](#5-запросы-к-базе-данных)
6. [Операции обновления данных](#6-операции-обновления-данных)
7. [Управление пользователями базы данных](#7-управление-пользователями-базы-данных)
8. [Экспорт и резервные копии базы](#8-экспорт-и-резервные-копии-базы)

## 1. Описание проекта

StreetBall App — это веб-приложение для организации и участия в уличных спортивных играх в Польше, с первоначальным акцентом на стритбол и баскетбол, с возможностью расширения на другие виды спорта (футбол, волейбол и т.д.). Главная цель приложения — упростить процесс поиска игр и площадок, а также организации собственных игр.

Система должна помочь энтузиастам уличных видов спорта находить доступные площадки, присоединяться к существующим играм и создавать собственные спортивные мероприятия. Изначально приложение фокусируется на играх типа стритбол и баскетбол, но структура данных была спроектирована с учетом будущего расширения на другие виды спорта.

## 2. Функциональные требования к базе данных

### Хранение информации о пользователях:
- Основные данные профиля (имя пользователя, электронная почта, пароль)
- Настройки уведомлений
- Ссылки на созданные и присоединенные игры

### Хранение информации о спортивных объектах:
- Географические координаты для отображения на карте
- Подробная информация о площадке (тип покрытия, освещение и т.д.)
- Поддерживаемые спортивные дисциплины
- Оценки и отзывы

### Хранение информации об игре:
- Время и продолжительность
- Местоположение (площадка)
- Формат игры (3х3, 5х5 и т.д.)
- Список участников
- Статус игры (запланирована, в процессе, завершена, отменена)

### Хранение уведомлений:
- Отслеживание прочитанных/непрочитанных уведомлений
- Связь с пользователями и играми
- Тип уведомления (напоминание, присоединение игрока и т.д.)

Кроме того, база данных должна обеспечивать:
- Поиск доступных площадок в окрестностях
- Фильтрацию игр по уровню подготовки, формату и виду спорта
- Мониторинг запланированных игр и напоминания о них
- Анализ активности пользователей и популярности площадок

## 3. Структура коллекций

База данных MongoDB состоит из следующих коллекций:

### Коллекция users:
Хранит информацию о пользователях приложения:
```json
{
  "_id": ObjectId,
  "username": String,
  "email": String,
  "password": String (hashed),
  "fullName": String,
  "avatar": String,
  "phone": String,
  "createdGames": [ObjectId], // ссылки на коллекцию games
  "joinedGames": [ObjectId],  // ссылки на коллекцию games
  "notifications": {
    "email": Boolean,
    "push": Boolean,
    "reminderTime": Number
  },
  "createdAt": Date,
  "updatedAt": Date
}
```

### Коллекция courts:
Хранит информацию о площадках, доступных в системе:
```json
{
  "_id": ObjectId,
  "name": String,
  "location": {
    "type": String,
    "coordinates": [Number, Number], // долгота и широта
    "address": String
  },
  "sportTypes": [String],
  "photos": [String],
  "description": String,
  "features": {
    "covered": Boolean,
    "lighting": Boolean,
    "surface": String,
    "changingRooms": Boolean
  },
  "workingHours": {
    "monday": { "open": String, "close": String },
    // ... для остальных дней недели
  },
  "rating": Number,
  "reviews": [
    {
      "user": ObjectId, // ссылка на коллекцию users
      "text": String,
      "rating": Number,
      "date": Date
    }
  ],
  "createdAt": Date,
  "updatedAt": Date
}
```

### Коллекция games:
Хранит информацию об играх, созданных пользователями:
```json
{
  "_id": ObjectId,
  "court": ObjectId, // ссылка на коллекцию courts
  "creator": ObjectId, // ссылка на коллекцию users
  "sportType": String,
  "dateTime": Date,
  "duration": Number, // в минутах
  "format": String, // например "3x3", "5x5"
  "maxPlayers": Number,
  "currentPlayers": [
    {
      "user": ObjectId, // ссылка на коллекцию users
      "joinedAt": Date
    }
  ],
  "status": String, // "scheduled", "in_progress", "completed", "cancelled"
  "description": String,
  "skillLevel": String, // "beginner", "intermediate", "advanced", "any"
  "isPrivate": Boolean,
  "inviteCode": String, // опционально для приватных игр
  "createdAt": Date,
  "updatedAt": Date
}
```

### Коллекция notifications:
Хранит информацию об уведомлениях, отправляемых пользователям:
```json
{
  "_id": ObjectId,
  "user": ObjectId, // ссылка на коллекцию users
  "game": ObjectId, // ссылка на коллекцию games
  "type": String, // тип уведомления
  "message": String,
  "isRead": Boolean,
  "scheduledFor": Date, // опционально для напоминаний
  "createdAt": Date,
  "updatedAt": Date
}
```

## 4. Реализация базы данных

Ниже представлены скрипты MongoDB, которые были использованы для создания и заполнения базы данных примерными данными.

### Создание коллекций и заполнение данными

Скрипт, создающий коллекции и заполняющий их примерными данными:

```javascript
// Переключение на схему или её создание
use streetball-db

// Удаление коллекций (если существуют)
db.users.drop()
db.courts.drop()
db.games.drop()
db.notifications.drop()

// Создание коллекций
db.createCollection("users")
db.createCollection("courts")
db.createCollection("games")
db.createCollection("notifications")

// Добавление документов в коллекцию users
db.users.insertMany([
  {
    username: "adamkoszy",
    email: "adam@example.pl",
    password: "$2a$10$qnPGv0opAm70OETJCu6qXuIjbwKS7aSQi69ZcP2aYKk2Wj3j7Fq/i", // "password123"
    fullName: "Adam Kowalski",
    avatar: "default-avatar.png",
    phone: "+48 501 234 567",
    createdGames: [],
    joinedGames: [],
    notifications: {
      email: true,
      push: true,
      reminderTime: 60
    },
    createdAt: new Date(),
    updatedAt: new Date()
  },
  // ... остальные пользователи
])

// Сохранение ID пользователей для удобства работы
const users = db.users.find().toArray();
const userIds = users.map(user => user._id);

// Добавление данных в коллекцию courts
db.courts.insertMany([
  {
    name: "Park Jordana",
    location: {
      type: "Point",
      coordinates: [21.012228, 52.229676],
      address: "ul. Parkowa 15, Warszawa"
    },
    sportTypes: ["streetball", "basketball"],
    photos: ["jordan_park_1.jpg", "jordan_park_2.jpg"],
    // ... остальные данные площадки
  },
  // ... остальные площадки
])

// ... остальные коллекции (games, notifications)
```

Скрипт создает четыре коллекции и заполняет их примерными данными, которые позволяют тестировать функциональность приложения. Добавлены примерные данные:
- 5 пользователей
- 5 площадок с различными характеристиками (покрытие, освещение и т.д.)
- 5 игр различных форматов (3х3, 5х5 и т.д.) и уровней подготовки
- 5 уведомлений различных типов

## 5. Запросы к базе данных

Ниже представлены примеры запросов, выполняемых в базе данных, которые реализуют требуемую функциональность приложения:

### Запрос 1: Поиск площадок, поддерживающих определенные виды спорта
```javascript
// Площадки для баскетбола и волейбола
db.courts.find({
  sportTypes: { $all: ["streetball", "volleyball"] }
})
```
Этот запрос возвращает площадки, которые поддерживают как стритбол, так и волейбол.

### Запрос 2: Поиск игр определенного формата в конкретный день
```javascript
// Игры 3х3, запланированные на завтра
const tomorrow = new Date();
tomorrow.setDate(tomorrow.getDate() + 1);
tomorrow.setHours(0, 0, 0, 0);
const dayAfter = new Date(tomorrow);
dayAfter.setDate(dayAfter.getDate() + 1);

db.games.find({
  format: "3x3",
  dateTime: { $gte: tomorrow, $lt: dayAfter },
  status: "scheduled"
})
```
Этот запрос ищет все запланированные игры в формате 3х3, которые проходят на следующий день.

### Запрос 3: Поиск площадок с определенными характеристиками
```javascript
// Крытые площадки с деревянным полом
db.courts.find({
  "features.covered": true,
  "features.surface": "wood"
})
```
Этот запрос ищет все крытые площадки с деревянным покрытием.

### Запрос 4: Поиск площадок с высокой оценкой и с отзывами
```javascript
// Площадки с оценкой > 4.5 и имеющие отзывы
db.courts.find({
  rating: { $gt: 4.5 },
  "reviews.0": { $exists: true }
})
```
Этот запрос возвращает площадки с оценкой выше 4.5, которые имеют хотя бы один отзыв.

### Запрос 5: Поиск пользователей с непрочитанными уведомлениями
```javascript
// Пользователи, у которых есть непрочитанные уведомления
const userIds = db.notifications.distinct("user", { isRead: false });
db.users.find({ _id: { $in: userIds } })
```
Этот запрос находит всех пользователей, у которых есть непрочитанные уведомления.

### Запрос 6: Агрегация - Количество запланированных игр на площадках
```javascript
// Подсчет запланированных игр для площадок
db.games.aggregate([
  { $match: { status: "scheduled" } },
  { $group: { 
    _id: "$court", 
    gameCount: { $sum: 1 } 
  }},
  { $lookup: {
    from: "courts",
    localField: "_id",
    foreignField: "_id",
    as: "courtInfo"
  }},
  { $project: {
    courtName: { $arrayElemAt: ["$courtInfo.name", 0] },
    gameCount: 1,
    _id: 0
  }},
  { $sort: { gameCount: -1 } }
])
```
Эта агрегация считает запланированные игры для каждой площадки и возвращает площадки, отсортированные по количеству игр.

### Запрос 7: Агрегация - Средний уровень заполнения игр по форматам
```javascript
// Подсчет СРЕДНЕГО количества людей для формата игр
db.games.aggregate([
  { $match: { status: "scheduled" } },
  { $project: {
    format: 1,
    occupancyRate: { 
      $multiply: [
        { $divide: [{ $size: "$currentPlayers" }, "$maxPlayers"] },
        100
      ]
    }
  }},
  { $group: {
    _id: "$format",
    averageOccupancy: { $avg: "$occupancyRate" },
    gameCount: { $sum: 1 }
  }},
  { $sort: { averageOccupancy: -1 } }
])
```
Эта агрегация вычисляет средний уровень заполнения (в процентах) для игр в различных форматах.

### Запрос 8: Агрегация - Анализ активности пользователей
```javascript
// Анализ активности пользователей: количество созданных игр и игр, к которым они присоединились
db.users.aggregate([
  { $project: {
    username: 1,
    createdGamesCount: { $size: "$createdGames" },
    joinedGamesCount: { $size: "$joinedGames" }
  }},
  { $addFields: {
    totalActivity: { $add: ["$createdGamesCount", "$joinedGamesCount"] }
  }},
  { $sort: { totalActivity: -1 } }
])
```
Эта агрегация анализирует активность пользователей, подсчитывая количество созданных и присоединенных игр, а затем сортирует пользователей по общей активности.

### Запрос 9: Агрегация - Количество площадок для каждого вида спорта
```javascript
// Количество площадок для каждого вида спорта
db.courts.aggregate([
  { $unwind: "$sportTypes" },
  { $group: {
    _id: "$sportTypes",
    courtCount: { $sum: 1 }
  }},
  { $sort: { courtCount: -1 } }
])
```
Эта агрегация считает, сколько площадок поддерживает каждый из видов спорта.

### Запрос 10: Агрегация - Анализ наиболее популярных часов игры
```javascript
// Анализ наиболее популярных часов
db.games.aggregate([
  { $match: { status: "scheduled" } },
  { $project: {
    hour: { $hour: "$dateTime" },
    dayOfWeek: { $dayOfWeek: "$dateTime" },
    players: { $size: "$currentPlayers" }
  }},
  { $group: {
    _id: { hour: "$hour", dayOfWeek: "$dayOfWeek" },
    gameCount: { $sum: 1 },
    totalPlayers: { $sum: "$players" }
  }},
  { $sort: { totalPlayers: -1 } },
  { $limit: 5 },
  { $project: {
    _id: 0,
    hour: "$_id.hour",
    dayOfWeek: "$_id.dayOfWeek",
    gameCount: 1,
    totalPlayers: 1,
    averagePlayers: { $divide: ["$totalPlayers", "$gameCount"] }
  }}
])
```
Эта агрегация анализирует, какие часы и дни недели наиболее популярны для игры, основываясь на количестве игроков.

## 6. Операции обновления данных

Ниже представлены примеры операций обновления данных в базе:

### Обновление 1: Изменение характеристик площадки
```javascript
// Изменение часов работы, добавление освещения и изменение описания
db.courts.updateOne(
  { name: "Park Jordana" },
  {
    $set: {
      "features.lighting": true,
      "workingHours.friday.open": "07:00",
      "workingHours.friday.close": "23:00",
      "description": "Обновленный парк с профессиональной баскетбольной площадкой, отличным покрытием и современным LED-освещением"
    }
  }
)
```
Эта операция обновляет часы открытия, добавляет освещение и меняет описание площадки "Park Jordana".

### Обновление 2: Добавление отзыва и обновление оценки площадки
```javascript
// Увеличение оценки площадки и добавление нового отзыва
const userId = db.users.findOne({ username: "piotr_pro" })._id;

db.courts.updateOne(
  { name: "Szkoła Podstawowa nr 45" },
  {
    $inc: { rating: 0.2 },
    $push: {
      reviews: {
        user: userId,
        text: "После ремонта покрытие значительно лучше, рекомендую для тренировок!",
        rating: 5,
        date: new Date()
      }
    }
  }
)
```
Эта операция добавляет новый отзыв о площадке и увеличивает ее оценку на 0.2.

### Обновление 3: Изменение статуса игры и удаление игрока
```javascript
// Изменение статуса игры и удаление игрока
db.games.updateOne(
  { sportType: "streetball", format: "3x3", status: "scheduled" },
  {
    $set: { status: "in_progress" },
    $pull: {
      currentPlayers: { 
        user: db.users.findOne({ username: "kasia_basket" })._id 
      }
    }
  }
)
```
Эта операция меняет статус игры на "в процессе" и удаляет одного из игроков из списка участников.

### Обновление 4: Добавление нового вида спорта и изменение названия поля
```javascript
// Добавление нового вида спорта и изменение названия поля
db.courts.updateOne(
  { name: "Centrum Młodzieżowe" },
  {
    $addToSet: { sportTypes: "football" },
    $rename: { "features.covered": "features.isIndoor" }
  }
)
```
Эта операция добавляет новый вид спорта (футбол) на площадку и меняет название поля с "covered" на "isIndoor".

### Обновление 5: Обновление нескольких документов одновременно
```javascript
// Изменение настроек уведомлений для всех пользователей и увеличение времени уведомления
db.users.updateMany(
  { "notifications.reminderTime": { $lt: 60 } },
  {
    $set: { "notifications.push": true },
    $inc: { "notifications.reminderTime": 15 }
  }
)
```
Эта операция обновляет настройки уведомлений для всех пользователей, у которых время напоминания меньше 60 минут, устанавливая push-уведомления на true и увеличивая время напоминания на 15 минут.

## 7. Управление пользователями базы данных

В рамках защиты базы данных созданы три пользователя с различными уровнями прав:

### Администратор базы данных
```javascript
// Админ для всей БД
use admin
db.createUser(
  {
    user: "dbAdmin",
    pwd: "secure_admin_password",
    roles: [
      { role: "userAdminAnyDatabase", db: "admin" },
      { role: "dbAdminAnyDatabase", db: "admin" },
      { role: "readWriteAnyDatabase", db: "admin" }
    ]
  }
)
```
Этот пользователь имеет полные права на управление базой данных, включая создание, изменение и удаление коллекций и документов в любой базе.

### Пользователь с правами на чтение и запись
```javascript
// Пользователь read/write только для streetball-db
use streetball-db
db.createUser(
  {
    user: "streetball_dev",
    pwd: "dev_password",
    roles: [
      { role: "readWrite", db: "streetball-db" }
    ]
  }
)
```
Этот пользователь может читать и изменять данные в базе streetball-db, но не имеет административных прав.

### Пользователь только для чтения
```javascript
// Пользователь read только для streetball_db
use streetball-db
db.createUser(
  {
    user: "streetball_reader",
    pwd: "read_only_password",
    roles: [
      { role: "read", db: "streetball-db" }
    ]
  }
)
```
Этот пользователь может только читать данные из базы streetball-db, без возможности вносить изменения.

## 8. Экспорт и резервные копии базы

Выполнен экспорт данных из базы с целью защиты от потери данных:

### Экспорт данных
```bash
mongodump --db streetball-db --out exported_collections
```
Команда экспортирует все коллекции из базы streetball-db в каталог exported_collections.

### Создание резервной копии
```bash
mongodump --db streetball-db --out streetball-db
```
Команда создает резервную копию базы streetball-db.

## Заключение

База данных StreetBall App была спроектирована и реализована в соответствии с функциональными требованиями. Она содержит все необходимые коллекции и связи между ними, что позволяет эффективно управлять данными приложения.

Проведенные операции и запросы подтверждают, что база данных отвечает всем требованиям относительно:
- Хранения информации о пользователях, площадках, играх и уведомлениях
- Поиска и фильтрации данных по различным критериям
- Анализа данных с помощью агрегации
- Обновления и модификации данных
- Защиты базы через систему прав пользователей
- Создания резервных копий и экспорта данных

Структура базы является гибкой и позволяет будущее расширение на дополнительные виды спорта и функциональность, в соответствии с требованиями проекта.