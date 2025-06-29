## Изменение набора IP-префиксов VPC в Routing Instance {#ri-ip-change}

Изменение набора IP-префиксов в приватном соединении сводится к следующим действиям:
1. Удаление уже существующих IP-префиксов из приватного соединения. 
2. Добавление новых IP-префиксов в приватное соединение.

Чтобы изменить набор IP-префиксов в приватном соединении, создайте [новое обращение в поддержку]({{ link-console-support }}).

### Обращение в поддержку для изменения набора IP-префиксов в приватном соединении {#prefix-change}

Обращение должно быть оформлено следующим образом:

```s
Тема: [CIC] Изменить набор IP-префиксов в приватном соединении.

Текст обращения:
Прошу изменить набор IP-префиксов для приватного соединения 
prc_id: cf3qdug4fsf737******

1. Удалить следующие IP-префиксы из указанных сетей (vpc_net_id):
 
 vpc_net_id: enpdffqsg8r221******
   {{ region-id }}-a: 10.60.192.0/21
   {{ region-id }}-b: 10.60.200.0/21, 10.60.220.0/24
   {{ region-id }}-d: 10.60.208.0/20

2. Добавить следующие IP-префиксы в указанные сети (vpc_net_id):

 vpc_net_id: enpdffqsg8r221******
   {{ region-id }}-a: 10.60.1.0/24
   {{ region-id }}-b: 10.60.4.0/24, 10.60.8.0/24
   {{ region-id }}-d: 10.60.10.0/20

 vpc_net_id: enpdhpdcp2u748******
   {{ region-id }}-a: 10.45.11.0/21, 10.45.100.0/24
   {{ region-id }}-b: 10.45.21.0/21
   {{ region-id }}-d: 10.45.31.0/20
```



где:

* `vpc_net_id` — идентификатор виртуальной сети {{ vpc-full-name }} в которую вносятся изменения. Для каждой виртуальной сети указывается список IPv4-префиксов подсетей, сгруппированных по [зонам доступности](../../overview/concepts/geo-scope.md) {{ yandex-cloud }}.


### Обращение в поддержку для добавления набора IP-префиксов в приватное соединение {#prefix-add}

Обращение должно быть оформлено следующим образом:

```s
Тема: [CIC] Добавить набор IP-префиксов в приватное соединение.

Текст обращения:
Прошу добавить набор IP-префиксов в приватное соединение
prc_id: cf3qdug4fsf737******

Добавить следующие IP-префиксы в указанные сети (vpc_net_id):

 vpc_net_id: enpdffqsg8r221******
   {{ region-id }}-a: 10.60.1.0/24
   {{ region-id }}-b: 10.60.4.0/24, 10.60.8.0/24
   {{ region-id }}-d: 10.60.10.0/20

 vpc_net_id: enpdhpdcp2u748******
   {{ region-id }}-a: 10.45.11.0/21, 10.45.100.0/24
   {{ region-id }}-b: 10.45.21.0/21
   {{ region-id }}-d: 10.45.31.0/20
```


где:

* `vpc_net_id` — идентификатор виртуальной сети {{ vpc-full-name }} в которую вносятся изменения. Для каждой виртуальной сети указывается список IPv4-префиксов подсетей, сгруппированных по [зонам доступности](../../overview/concepts/geo-scope.md) {{ yandex-cloud }}.


### Обращение в поддержку для удаления набора IP-префиксов из приватного соединения {#prefix-del}

Обращение должно быть оформлено следующим образом:

```s
Тема: [CIC] Удалить набор IP-префиксов из приватного соединения.

Текст обращения:
Прошу удалить набор IP-префиксов из приватного соединения
prc_id: cf3qdug4fsf737******

1. Удалить следующие IP-префиксы из указанных сетей (vpc_net_id):

 vpc_net_id: enpdffqsg8r221******
   {{ region-id }}-a: 10.60.192.0/21
   {{ region-id }}-b: 10.60.200.0/21, 10.60.220.0/24
   {{ region-id }}-d: 10.60.208.0/20

 vpc_net_id: enpdffqsg8r221******
   {{ region-id }}-a: 10.60.1.0/24
   {{ region-id }}-b: 10.60.4.0/24, 10.60.8.0/24
   {{ region-id }}-d: 10.60.10.0/20
```


где:

* `vpc_net_id` — идентификатор виртуальной сети {{ vpc-full-name }} в которую вносятся изменения. Для каждой виртуальной сети указывается список IPv4-префиксов подсетей, сгруппированных по [зонам доступности](../../overview/concepts/geo-scope.md) {{ yandex-cloud }}.


### Ответ поддержки по обращению клиента {#priv-ticket-resp}

Пример ответа службы поддержки на обращение об изменении IP-префиксов в приватном соединении (для информации):

```s
Изменения набора IP-префиксов для приватного соединения приняты.
prc_id: cf3qdug4fsf737******
```

{% note info %}

Изменение набора IP-префиксов может занимать до двух рабочих дней. Поддержка дополнительно уведомит вас о завершении процесса.  

{% endnote %}

При возникновении проблем с IP-связностью свяжитесь с поддержкой для их диагностики и устранения.

