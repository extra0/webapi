# API для webapp в Telegram

- [Шаблон запроса](#шаблон-запроса)
- [Доступные классы](#classes)
- [Универсальные методы](#methods)
- [Ошибки](#errors)

API разработана для обмена и синхронизации данных между проектом Х10 Движение и webapp Telegram.

## <a id="template">Шаблон запроса</a>

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

## <a id="classes">Доступные классы</a>

## <a id="methods">Универсальные методы</a>

## <a id="errors">Ошибки</a> 

При указании наименования класса
