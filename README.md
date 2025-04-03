# API для webapp в Telegram

- [Шаблон запроса](#шаблон-запроса)
- [Доступные классы {className}](#доступные-классы)
- [Универсальные методы](#универсальные-методы)
- [Классы](#классы)


API разработана для синхронизации данных между проектом Х10 Движение и webapp Telegram.

## Шаблон запроса

```https
  POST /ajax/webapp-api/{className}/{methodName}
```

| Parameter    | Type     | Description                   |
| :----------- | :------- | :---------------------------- |
| `className`  | `string` | **Required**. Название класса |
| `methodName` | `string` | **Required**. Название метода |

При отправке запроса в header передаем значение токена авторизации и формата данных

```
headers: {
  'Authorization': {authToken},
  'Content-Type': 'application/json'
}
```

Ответ возвращаются в JSON формате.

### Первичная валидация запроса
* Проверка токена авторизации {authToken}
* Наличие предустановленного в API класса {className}
* Наличие метода в классе {methodName}

### Success response
```
{
  'success': true,
  'data': {data}
}
```

### Error response
```
{
  'success': false,
  'error': {errorText},
}
```

## Доступные классы
| Class    |  Description                   |  |
| :----------- | :---------------------------- | :------------ |
| `user`  | Работа с сущностью пользователя | [Подробнее](#пользователь) |
| `image`  | Работа с изображениями | [Подробнее](#сохранение-изображения) |
| `academy` | Работа с инфоблоком "Академия и проекты" | [Подробнее](#) |
| `clamp` | Работа с инфоблоком "Клампы" | [Подробнее](#) |
| `event` | Работа с инфоблоком "Мероприятия пользователя" | [Подробнее](#) |

Список классов дополняется при необходимости масштабирования webapi.

## Универсальные методы
### 📋 Получение списка элементов

```https
  POST /ajax/webapp-api/{className}/getList
```

| Payload      | Type     | Description                   |
| :----------- | :------- | :---------------------------- |
| `fields`  | `object` | Возвращаемые поля. ([Подробнее](https://dev.1c-bitrix.ru/api_help/iblock/classes/ciblockelement/getlist.php)) |
| `properties`  | `object` | Свойства элементов (уникальные для каждого класса)|
| `order` | `string` | Поле сортировки (default: id) |
| `sort` | `string` | Направление сортировки (asc или desc (default)) |
| `filter` | `object` | Поля для фильтрации |
| `offset` | `number` | смещение на определенное кол-во элементов в выборке |
| `limit` | `number` | ограничение по кол-ву элементов. Max: 100, min: 1. (default: 10)  |

Все параметры передаются в camelCase стиле

#### 📦 Payload example
```
{
  'fields': ['id', 'name', 'previewText', 'detailText', 'createdBy'],
  'properties': ['status', 'otherProp'],
  'order': 'name',
  'sort': 'asc',
  'filter': {
    createdBy: 12300, - ID пользователя
    id: 100500, - ID элемента
    ...
    propertyStatus: 1, - фильтрация по кастомному свойству 'status'
    propertyGroupId: 1, - фильтрация по кастомному свойству 'groupId'
    ...
  },
  'offset': 0,
  'limit': 10,
}
```

#### 📩 Response example
```
{
  'success': true,
  'data': [items array with choosen fields] or [],
}
```

<hr>

### 🔂 Получение информации об элементе

```https
  POST /ajax/webapp-api/{className}/getById
```

Метод основан на getList с обязательным параметром в фильтре "id"

| Payload      | Type     | Description                   |
| :----------- | :------- | :---------------------------- |
| `fields`  | `object` | Возвращаемые поля. ([Подробнее](https://dev.1c-bitrix.ru/api_help/iblock/classes/ciblockelement/getlist.php)) |
| `properties`  | `object` | Свойства элементов (уникальные для каждого класса)|
| `filter` | `object` | Поля для фильтрации. **Required** id |

#### 📦 Payload example
```
{
  'fields': ['id', 'name', 'previewText', 'detailText', 'createdBy'],
  'properties': ['status', 'otherProp'],
  'filter': {
    id: 100500, - ID элемента (обязательный параметр)
    ...
    propertyStatus: 1, - фильтрация по кастомному свойству 'status'
    propertyGroupId: 1, - фильтрация по кастомному свойству 'groupId'
    ...
  },
}
```

#### 📩 Response example
```
{
  'success': true,
  'data': [data],
}
```

<hr>

### ➕ Создание элемента

```https
  POST /ajax/webapp-api/{className}/create
```

Метод создает новый элемент в выбранном классе. 

| Payload      | Type     | Description                   |
| :----------- | :------- | :---------------------------- |
| `userId`  | `string / number` | **Required** ID пользователя |
| `fields`  | `object` | **Required** Поля с данными элемента |
| `properties`  | `object` | Поля доп свойствами для элемента (optional) |


#### 📦 Payload example
```
{
  'userId': 100500,
  'fields': {
    'name': 'Element name',
    'detailText': 'Lorem....',
    'previewText': 'Lorem.... Preview',
    'previewPicture': 100500 - ID изображения на сервере (получаем при помощи класса SaveFile),
    ...
  }
  'properties': {
    'status': 10,
    'groupId': 20,
    ...
  },
}
```

#### 📩 Response example
```
{
  'success': true,
  'data': {
    'elementId': 100500 - ID созданного элемента,
  }
}
```

<hr>

### 📝 Редактирование элемента

```https
  POST /ajax/webapp-api/{className}/update
```

Метод редактирует элемент в выбранном классе.<br>
Перед редактированием производится проверка на авторство элемента (userId === createdBy). createdBy не передается

| Payload      | Type     | Description                   |
| :----------- | :------- | :---------------------------- |
| `userId`  | `string / number` | **Required** ID пользователя |
| `elementId`  | `string / number` | **Required** ID элемента |
| `fields`  | `object` | Поля с данными элемента (optional) |
| `properties`  | `object` | Поля доп свойствами для элемента (optional) |

#### 📦 Payload example
```
{
  'userId': 100500,
  'elementId': 100500,
  'fields': {
    'name': 'Element name',
    'detailText': 'Lorem....',
    'previewText': 'Lorem.... Preview',
    'previewPicture': 100500 - ID изображения на сервере (получаем при помощи класса SaveFile),
    ...
  }
  'properties': {
    'status': 10,
    'groupId': 20,
    ...
  },
}
```

#### 📩 Response example
```
{
  'success': true,
  'data': null,
}
```

<hr>

### ❌ Удаление элемента

```https
  POST /ajax/webapp-api/{className}/delete
```

Удаление выбранного элемента.<br>
По факту выполняется деактивация элемента в БД с последующей возможностью активировать через админов.<br>
Перед удалением производится проверка на авторство элемента (userId === createdBy). createdBy не передается

| Payload      | Type     | Description                   |
| :----------- | :------- | :---------------------------- |
| `userId`  | `string / number` | **Required** ID пользователя |
| `elementId`  | `string / number` | **Required** ID элемента |

#### 📦 Payload example
```
{
  'userId': 100500,
  'elementId': 100500,
}
```

#### 📩 Response example
```
{
  'success': true,
  'data': null,
}
```
<br>
<br>
<hr>
<br>

## Классы
## 👤 Пользователь
* authorization - авторизация
* registration - регистрация
* update - обновление **(в разработке)**
* getById - получение информации по ID пользователя **(в разработке)**

### Авторизация пользователя

```https
  POST /ajax/webapp-api/user/authorization
```
| Payload      | Type     | Description                   |
| :----------- | :------- | :---------------------------- |
| `login`  | `string` | **Required** login (email) пользователя |
| `password`  | `string` | **Required** пароль пользователя (min lenght: 8) |

#### 📦 Payload example
```
{
  'login': 'example@mail.ru',
  'password': '123123123',
}
```

#### 📩 Response example
```
{
  'success': true,
  'userId': 100500 (int),
}
```
<hr>

### Регистрация пользователя

```https
  POST /ajax/webapp-api/user/registration
```
| Payload      | Type     | Description                   |
| :----------- | :------- | :---------------------------- |
| `email`  | `string` | **Required** email |
| `password`  | `string` | **Required** пароль (min lenght: 8) |
| `name`  | `string` | **Required** Имя |
| `lastName`  | `string` | **Required** Фамилия |
| `phone`  | `string` | **Required** Телефон (в международном формате +7 (985) 123-45-67) |
| `city`  | `string` | **Required** Город пользователя |
| `telegram`  | `string` | **Required** Телеграм пользователя (@example or https://t.me/example) |
| `role`  | `number` | **Required** Роль при регистрации (7 - участник (default), 8 - клампер) |

#### 📦 Payload example
```
{
  'email': 'example@mail.ru',
  'password': '123123123',
  'name': 'name',
  'lastName': 'lastName',
  'phone': '+7 (985) 123-45-67',
  'city': 'Москва',
  'telegram': '@example',
  'role': 7,
}
```

#### 📩 Response example
```
{
  'success': true,
  'userId': 100500 (int),
}
```

### Обновление пользователя (в разработке)

```https
  POST /ajax/webapp-api/user/update
```
<hr>
<br>

### 📸 Сохранение изображения

```https
  POST /ajax/webapp-api/image/getTmpImageId
```

Метод отправляет на сервер и сохраняет изображение для последующего переиспользования при создании или редактировании сущности.<br>
Max size: 15 Mb

| Payload      | Type     | Description                   |
| :----------- | :------- | :---------------------------- |
| `file`  | `binary` | **Required** Фото пользователя. На бекенде будет получать данные так $request->getUploadedFiles() |

#### 📦 Payload example
```
{
  'file': (binary)
}
```

#### 📩 Response example
```
{
  'success': true,
  'data': [
    'id': 100500 - ID для переиспользования (int)
    'name': 'image name', (string)
    'src': 'src path', (string)
    'size': 'size name', (int)
  ],
}
```

## Академия и проекты (academy)
Метод использует [стандартные поля](https://dev.1c-bitrix.ru/api_help/iblock/classes/ciblockelement/getlist.php) данных.<br>
Ниже описаны кастомные свойства и обязательные поля для класса

| Property name      | Type     | Description                   |
| :----------- | :------- | :---------------------------- |
| `userId`  | `number` | Привязка к пользователю |
| `position`  | `string` | Должность |
<hr>

## Клампы (clamp)
Метод использует [стандартные поля](https://dev.1c-bitrix.ru/api_help/iblock/classes/ciblockelement/getlist.php) данных.<br>
Ниже описаны кастомные свойства и обязательные поля для класса

| Common fields      | Type     | Description                   |
| :----------- | :------- | :---------------------------- |
| `name`  | `string` | **Required** Название. Назначается на основе Фамилии / имени и текущего кол-ва активных клампов |
| `previewPicture`  | `number` | **Required** ID изображения. Картинка клампа |
| `previewText`  | `string` | Слоган клампа |
| `detailText`  | `string` | Описание клампа |

<br>

| Property name      | Type     | Description                   |
| :----------- | :------- | :---------------------------- |
| `city`  | `string` | **Required** Город |
| `groupDailing`  | `number` | Набор в кламп<br>21 - Идет набор (default)<br>22 - Набор закрыт |
| `chat`  | `string` | Чат клампа в соц. сетях |
<hr>

## Мероприятия пользователя (event)
Метод использует [стандартные поля](https://dev.1c-bitrix.ru/api_help/iblock/classes/ciblockelement/getlist.php) данных.<br>
Ниже описаны кастомные свойства и обязательные поля для класса

| Common fields      | Type     | Description                   |
| :----------- | :------- | :---------------------------- |
| `name`  | `string` | **Required** Название мероприятия |
| `previewPicture`  | `number` | **Required** Фото мероприятия |
| `previewText`  | `string` | **Required** Краткое описание |
| `detailText`  | `string` | **Required** Подробное описание |

<br>

| Property name      | Type     | Description                   |
| :----------- | :------- | :---------------------------- |
| `city`  | `string` | **Required** Город |
| `format`  | `string` | Формат мероприятия |
| `dateEvent`  | `date:YY-mm-dd` | Дата мероприятия |
| `timeEvent`  | `string` | Время мероприятия |
| `placeEvent`  | `string` | Место проведения |
| `contactPerson`  | `string` | Контактное лицо |
| `contactPersonPhone`  | `string` | Контактное лицо (Телефон) |
| `userRegistrationList`  | `string` | Зарегистрированные пользователи |
