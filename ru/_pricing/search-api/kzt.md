| Услуга  | Цена за 1000 запросов,<br/>вкл. НДС |
|---------|-------------------------------------|
| **Запросы через [API v1](../../search-api/concepts/index.md#api-v1)** | |
| Ночные синхронные запросы, первые 1000 запросов в месяц | {{ sku|KZT|searchapi.requests.night.v1|string }} |
| Ночные синхронные запросы, свыше 1000 запросов в месяц | {{ sku|KZT|searchapi.requests.night.v1|pricingRate.1|string }} |
| Дневные синхронные запросы | {{ sku|KZT|searchapi.requests.day.v1|string }} |
| **Запросы через [API v2](../../search-api/concepts/index.md#api-v2)** | |
| Синхронные запросы | {{ sku|KZT|searchapi.requests.sync.v3|string }} |
| Отложенные запросы | {{ sku|KZT|searchapi.requests.async.v3|string }} | 
| Синхронный запрос с [генеративным ответом](../../search-api/concepts/generative-response.md) | {% calc [currency=KZT] {{ sku|KZT|searchapi.generative.requests.v3|number }} × 1000 %} |