# {{ sws-full-name }} overview

{% include [about-sws](../../_includes/smartwebsecurity/about-sws.md) %}

## How it works {#how-it-works}

{{ sws-name }} checks the HTTP requests sent to the protected resource via the virtual host of the L7 load balancer against the [rules](rules.md) configured in the [security profile](profiles.md). Depending on the results of the check, the requests are routed to the virtual host, blocked, or sent to [{{ captcha-full-name }}](../../smartcaptcha/) for additional verification.

![schema](../../_assets/smartwebsecurity/schema.svg)

{% include [realized-waf-concept](../../_includes/smartwebsecurity/realized-waf-concept.md) %}

{% include [realized-arl-concept](../../_includes/smartwebsecurity/realized-arl-concept.md) %}

## Monitoring and audit {#monitoring-audit}

{{ sws-name }} logs are sent to [{{ cloud-logging-full-name }}](../../logging/).

{{ sws-name }} metrics are sent to [{{ monitoring-full-name }}](../../monitoring/).

{{ sws-name }} audit logs are sent to [{{ at-full-name }}](../../audit-trails/).

{% include [user-data-to-ml](../../_includes/smartwebsecurity/user-data-to-ml.md)%}

## {{ alb-name }} coniguration recommendations {#alb-settings-recommendation}

{% include [alb-settings-recommendation](../../_includes/smartwebsecurity/alb-settings-recommendation.md) %}
