Добавьте ресурсную [А-запись](../../dns/concepts/resource-record.md#a) в публичную DNS-зону вашего домена, указав в ней значения:

* `Имя записи` — адрес вашего домена, заканчивающийся на точку. Например: `example.com.` или `my.first.example.com.`.
* `Значение` — полученный на предыдущем шаге IPv4-адрес прокси-сервера.

Эта запись перенаправляет запросы, которые поступают на ваш домен, на IP-адрес прокси-сервера.

{% include [create-record-instruction-notice](../../_includes/dns/create-record-instruction-notice.md) %}