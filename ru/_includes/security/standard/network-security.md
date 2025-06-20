# Требования к сетевой безопасности

## 2. Сетевая безопасность {#network-security}


В этом разделе представлены рекомендации пользователям по настройкам безопасности в [{{ vpc-full-name }}](../../../vpc/).


Подробно о том, как настроить сетевую инфраструктуру, рассказывается в вебинаре [Как работает сеть в {{ yandex-cloud }}](https://www.youtube.com/watch?v=g3cZ0o50qH0).


Чтобы изолировать приложения друг от друга, поместите ресурсы в разные [группы безопасности](../../../vpc/concepts/security-groups.md), а если требуется наиболее строгая изоляция — в разные [сети](../../../vpc/concepts/network.md#network). Трафик внутри сети по умолчанию разрешен, а между сетями — нет. Трафик между сетями можно передавать только через [виртуальную машину](../../../compute/concepts/vm.md) с двумя сетевыми интерфейсами в разных сетях, [VPN](../../../glossary/vpn.md) или сервис [{{ interconnect-full-name }}](../../../interconnect/index.yaml).

### Общее {#general}

#### 2.1 Для объектов облака используется межсетевой экран или группы безопасности {#firewall}

Встроенный механизм групп безопасности позволяет управлять доступом ВМ к ресурсам и группами безопасности {{ yandex-cloud }} или ресурсам в интернете. Группа безопасности — это набор правил для входящего и исходящего трафика, который можно назначить на сетевой интерфейс ВМ. Группы безопасности работают как stateful firewall, то есть отслеживают состояние сессий: если правило разрешает создать сессию, ответный трафик будет автоматически разрешен. Инструкцию по настройке групп безопасности см. в разделе [{#T}](../../../vpc/operations/security-group-create.md). Указать группу безопасности можно в настройках ВМ.

Группы безопасности могут использоваться для защиты:
* ВМ.
* [Управляемых баз данных](/services#data-platform).
* [Балансировщиков нагрузки](../../../application-load-balancer/concepts/application-load-balancer.md) [{{ alb-full-name }}](../../../application-load-balancer/).
* [Кластеров](../../../managed-kubernetes/concepts/index.md#kubernetes-cluster) [{{ managed-k8s-full-name }}](../../../managed-kubernetes/).

Список доступных сервисов расширяется.

Вы можете управлять сетевым доступом без групп безопасности, например, с помощью отдельной ВМ — межсетевой экран на основе образа [NGFW]({{ link-cloud-marketplace }}/products/usergate/ngfw) из {{ marketplace-full-name }}, либо своего собственного образа. Использование NGFW может быть критично для тех клиентов, которым необходима следующая функциональность:
* Составление логов сетевых соединений.
* Потоковый анализ трафика на предмет зловредного контента.
* Обнаружение сетевых атак по сигнатурам.
* Другая функциональность классических NGFW-решений.

Убедитесь, что в ваших [облаках](../../../resource-manager/concepts/resources-hierarchy.md#cloud) используется что-либо из списка:

* Группы безопасности на каждом объекте облака.
* Отдельная ВМ NGFW из {{ marketplace-name }}.
* Принцип [BYOI](https://en.wikipedia.org/wiki/Bring_your_own_operating_system), например: [собственный образ](../../../compute/operations/image-create/upload.md) диска.

{% list tabs group=instructions %}

- Проверка в консоли управления {#console}

  Проверка наличия групп безопасности на объектах:
  1. Откройте [консоль управления {{ yandex-cloud }}]({{ link-console-main }}) в вашем браузере.
  1. Перейдите в каждое облако и в каждый [каталог](../../../resource-manager/concepts/resources-hierarchy.md#folder) и последовательно открывайте все перечисленные ресурсы в пункте «Объекты, на которые возможно применить группы безопасности».
  1. В настройках объектов найдите параметр **Группа безопасности** и убедитесь, что назначена хотя бы одна группа безопасности.
  1. Если в параметрах каждого объекта, который поддерживает группы безопасности указана хотя бы одна группа, рекомендация выполняется. Если нет, перейдите к пункту «Инструкции и решения по выполнению».

  Проверка наличия NGFW вместо групп безопасности:
  1. Откройте консоль управления {{ yandex-cloud }} в вашем браузере.
  1. Перейдите в каждое облако и в каждый каталог и последовательно откройте все [диски](../../../compute/concepts/vm.md) ВМ.
  1. В настройках дисков найдите параметр **Продукт {{ marketplace-short-name }}**.
  1. Если в параметрах **Продукт {{ marketplace-short-name }}** в диске указано одно из названий продуктов NGFW: Check Point CloudGuard IaaS — Firewall & Threat Prevention PAYG, UserGate NGFW, рекомендация выполняется. Если нет, перейдите к пункту «Инструкции и решения по выполнению».

- Проверка через CLI {#cli}

  1. Посмотрите доступные вам организации и зафиксируйте необходимый `ID`:

     ```bash
     yc organization-manager organization list
     ```

  1. Выполните команду для поиска объектов облака без группы безопасности:

     ```bash
     export ORG_ID=<ID организации>
     for CLOUD_ID in $(yc resource-manager cloud list --organization-id=${ORG_ID} --format=json | jq -r '.[].id');
     do for FOLDER_ID in $(yc resource-manager folder list --cloud-id=$CLOUD_ID --format=json | jq -r '.[].id'); 
     do for VM_ID in $(yc compute instance list --folder-id=$FOLDER_ID --format=json | jq -r '.[].id'); do yc compute instance get --id=$VM_ID --format=json | jq -r '. | select(.network_interfaces[].security_group_ids | not)' | jq -r '.id'
     done;
     done;
     done
     ```

  1. Если выдается пустая строка, рекомендация выполняется. Если выдается результат с `ID` облачного ресурса, перейдите к пункту «Инструкции и решения по выполнению».

  Проверка наличия NGFW вместо группы безопасности:
  1. Выполните команду для поиска NGFW в облаке. По умолчанию команда ищет Checkpoint или Usergate. Если используете свой образ, укажите его.

     ```bash
     export ORG_ID=<ID организации>
     for CLOUD_ID in $(yc resource-manager cloud list --organization-id=${ORG_ID} --format=json | jq -r '.[].id');
     do for FOLDER_ID in $(yc resource-manager folder list --cloud-id=$CLOUD_ID --format=json | jq -r '.[].id');
     do for DISK_ID in $(yc compute disk list --folder-id=$FOLDER_ID --format=json | jq -r '.[].id'); do yc compute disk get --id=$DISK_ID --format=json | jq -r '. | select(.product_ids[0]=="f2ecl4ak62mjbl13qj5f" or .product_ids[0]=="f2eqc5sac8o5oic7m99k")' | jq -r '.id'
     done;
     done;
     done
     ```

  1. Если выдается `ID` ВМ с NGFW, рекомендация выполняется. Если выдается пустая строка, перейдите к пункту «Инструкции и решения по выполнению».

{% endlist %}

**Инструкции и решения по выполнению:**
* Примените группы безопасности на все объекты, на которых группа отсутствует.
* Для применения группы безопасности с помощью {{ TF }} используйте [настройку групп безопасности (dev/stage/prod) с помощью {{ TF }}](https://github.com/yandex-cloud/yc-solution-library-for-security/tree/master/network-sec/segmentation).
* Для использования NGFW [установите](https://github.com/yandex-cloud/yc-solution-library-for-security/tree/master/network-sec/checkpoint-1VM) на ВМ межсетевой экран (NGFW): Check Point.
* [Инструкция](https://docs.google.com/document/d/1yYwHorzkwXwIUGeG3n_K6Zo-07BVYowZJL7q2bAgVR8/edit?usp=sharing) по использованию UserGate NGFW в облаке.
* NGFW в режиме [active-passive](https://github.com/yandex-cloud/yc-solution-library-for-security/blob/master/network-sec/checkpoint-2VM_active-active/README.md).

#### 2.2 В {{ vpc-name }} создана группа безопасности и не используется группа безопасности по умолчанию {#vpc-sg}

*Группа безопасности* (Security Group, SG) — это ресурс, который создается на уровне [облачной сети](../../../vpc/concepts/network.md#network). После создания [группа безопасности](../../../vpc/concepts/security-groups.md) может использоваться в [сервисах](../../../vpc/concepts/security-groups.md#security-groups-apply) {{ yandex-cloud }} для разграничения сетевого доступа объекта, к которому она применяется.

*Группа безопасности по умолчанию* (Default Security Group, DSG) создается автоматически при создании [новой облачной сети](../../../vpc/concepts/network.md#network). Группа безопасности по умолчанию обладает следующими свойствами:

* В новой сети будет разрешать весь сетевой трафик в обоих направлениях — исходящий (egress) и входящий (ingress).
* Действует для трафика, проходящего через все подсети в сети, где она создана.
* Работает лишь в том случае, если на объект еще явно не назначена группа безопасности.
* Ее невозможно удалить, она автоматически удаляется вместе с удалением сети.

Группа безопасности по умолчанию — это удобный, но небезопасный механизм, который автоматически разрешает весь сетевой трафик (входящий и исходящий) для объектов в вашей сети. Хотя это упрощает начальную настройку, такая открытость создает серьезные риски:

* Злоумышленники могут получить доступ к ресурсам через публичные интерфейсы.
* Неконтролируемый трафик повышает уязвимость к DDoS-атакам и сканированию портов.
* DSG активна только до тех пор, пока вы не назначите объекту другую группу безопасности.

Рекомендуем [создать](../../../vpc/operations/security-group-create.md) собственную группу безопасности с [правилами](../../../vpc/concepts/security-groups.md#security-groups-rules), которые явно разрешают только нужный трафик (например, `HTTP/HTTPS` для веб-серверов или `SSH` для администрирования), и назначить созданную группу на облачные [объекты](../../../vpc/concepts/security-groups.md#security-groups-apply) ([виртуальные машины](../../../compute/concepts/vm.md), [кластеры {{ k8s }}](../../../managed-kubernetes/concepts/index.md#kubernetes-cluster) и т.д.), чтобы переопределить DSG.

Это важно, потому что без ваших правил облачные ресурсы остаются открытыми для любых подключений из интернета, а собственные группы безопасности позволяют реализовать [принцип минимальных привилегий](../../../iam/best-practices/using-iam-securely.md#restrict-access), снижая поверхность атак.

Группы безопасности можно комбинировать — на один объект можно назначить до пяти групп, что делает процесс разграничения доступа более гибким.

{% list tabs group=instructions %}

- Проверка в консоли управления {#console}

  1. Откройте консоль {{ yandex-cloud }} в вашем браузере.
  1. Перейдите в каждое облако, далее в каждый каталог и в каждую {{ vpc-name }}.
  1. Перейдите в раздел **Группы безопасности**.
  1. Если для каждой сети {{ vpc-name }} обнаружена как минимум одна группа безопасности в дополнение к группе безопасности по умолчанию, рекомендация выполняется. Если нет, перейдите к пункту «Инструкции и решения по выполнению».

- Проверка через CLI {#cli}

  1. Посмотрите доступные вам организации и зафиксируйте необходимый `ID`:

     ```bash
     yc organization-manager organization list
     ```

  1. Выполните команду для поиска каталогов без группы безопасности:

     ```bash
     export ORG_ID=<ID организации>
     for CLOUD_ID in $(yc resource-manager cloud list --organization-id=${ORG_ID} --format=json | jq -r '.[].id');
     do for FOLDER_ID in $(yc resource-manager folder list --cloud-id=$CLOUD_ID --format=json | jq -r '.[].id'); \
     do echo "SG_ID: " && yc vpc security-group list --folder-id=$FOLDER_ID --format=json | jq -r '.[] | select(.id)' | jq -r '.id' && echo "FOLDER_ID: " $FOLDER_ID && echo "-----"
     done;
     done
     ```

  1. Если у каждого сочетания `SG_ID` напротив `FOLDER_ID`, в которой она находится, указаны `ID`, рекомендация выполняется. Если нет, перейдите к пункту «Инструкции и решения по выполнению».

{% endlist %}

**Инструкции и решения по выполнению:**

Создайте группу безопасности в каждой {{ vpc-name }} с ограниченными правилами доступа, чтобы ее можно было назначать на облачные объекты.

#### 2.3 В группах безопасности отсутствует слишком широкое правило доступа {#access-rule}

В группе безопасности существует возможность открыть сетевой доступ для абсолютно всех IP-адресов интернета и также по всем диапазонам портов. Опасное правило выглядит следующим образом:
* Диапазон портов: 0-65535 или пусто.
* Протокол: любой или TCP/UDP.
* Источник: CIDR.
* CIDR блоки: 0.0.0.0/0 (доступ со всех адресов) или ::/0 (ipv6).

{% note warning %}

Если диапазон портов не указан, считается, что доступ предоставляется по всем портам (0-65535).

{% endnote %}

Открывать сетевой доступ необходимо только по тем портам, которые требуются для работы вашего приложения, и для тех адресов, с которых необходимо подключаться к вашим объектам.

{% list tabs group=instructions %}

- Проверка в консоли управления {#console}

  1. Откройте консоль {{ yandex-cloud }} в вашем браузере.
  1. Перейдите в каждое облако, далее в каждый каталог и в каждую {{ vpc-name }}.
  1. Перейдите в раздел **Группы безопасности**.
  1. Если не обнаружено ни одной группы безопасности, в которой есть правила сетевого доступа, разрешающие доступ по всем портам для всех адресов (интерпретация указана выше), рекомендация выполняется. Если нет, то перейдите к пункту «Инструкции и решения по выполнению».

- Проверка через CLI {#cli}

  1. Посмотрите доступные вам организации и зафиксируйте необходимый `ID`:

     ```bash
     yc organization-manager organization list
     ```

  1. Найдите группы безопасности с опасным правилом доступа:

     ```bash
     export ORG_ID=<ID организации>
     for CLOUD_ID in $(yc resource-manager cloud list --organization-id=${ORG_ID} --format=json | jq -r '.[].id');
     do for FOLDER_ID in $(yc resource-manager folder list --cloud-id=$CLOUD_ID --format=json | jq -r '.[].id'); \
     do echo "SG_ID: " && yc vpc security-group list --folder-id=$FOLDER_ID \
     --format=json | jq -r '.[] | select(.rules[].direction=="INGRESS" and .rules[].ports.to_port=="65535" and .rules[].cidr_blocks.v4_cidr_blocks[]=="0.0.0.0/0")' | jq -r '.id' \
     && echo "FOLDER_ID: " $FOLDER_ID && echo "-----"
     done;
     done
     ```

  1. Если `SG_ID` напротив `FOLDER_ID` указано пустое значение, рекомендация выполняется. Если вы видите не пустое `SG_ID`, перейдите к пункту «Инструкции и решения по выполнению».

{% endlist %}

**Инструкции и решения по выполнению:**

Удалите опасное правило в каждой группе безопасности или отредактируйте, указав доверенные IP-адреса.

#### 2.4 Доступ по управляющим портам открыт только для доверенных IP-адресов {#trusted-ip}

Рекомендуется открывать доступ к вашей облачной инфраструктуре по управляющим портам только с доверенных IP-адресов. Убедитесь, что в ваших правилах доступа в рамках группы безопасности отсутствуют широкие правила доступа по управляющим портам:
* Диапазон портов: 22, 3389 или 21.
* Протокол: TCP.
* Источник: CIDR.
* CIDR блоки: 0.0.0.0/0 (доступ со всех адресов) или ::/0 (ipv6).

{% list tabs group=instructions %}

- Проверка в консоли управления {#console}

  1. Откройте консоль {{ yandex-cloud }} в вашем браузере.
  1. Перейдите в каждое облако, далее в каждый каталог и в каждую {{ vpc-name }}.
  1. Перейдите в раздел **Группы безопасности**.
  1. Если не обнаружено ни одной группы безопасности, в которой есть правила сетевого доступа, разрешающие доступ по управляющим портам для всех адресов (интерпретация указана выше), рекомендация выполняется. Если нет, перейдите к пункту «Инструкции и решения по выполнению».

- Проверка через CLI {#cli}

  1. Посмотрите доступные вам организации и зафиксируйте необходимый `ID`:

     ```bash
     yc organization-manager organization list
     ```

  1. Выполните команду для поиска групп безопасности с опасным правилом доступа:

     ```bash
     export ORG_ID=<ID организации>
     for CLOUD_ID in $(yc resource-manager cloud list --organization-id=${ORG_ID} --format=json | jq -r '.[].id');
     do for FOLDER_ID in $(yc resource-manager folder list --cloud-id=$CLOUD_ID --format=json | jq -r '.[].id'); \
     do echo "SG_ID: " && yc vpc security-group list --folder-id=$FOLDER_ID \
     --format=json | jq -r '.[] | select(.rules[].direction=="INGRESS" and (.rules[].ports.to_port=="22" or .rules[].ports.to_port=="3389" or .rules[].ports.to_port=="21") and .rules[].cidr_blocks.v4_cidr_blocks[]=="0.0.0.0/0")' | jq -r '.id' \
     && echo "FOLDER_ID: " $FOLDER_ID && echo "-----"
     done;
     done
     ```

  1. Если `SG_ID` напротив `FOLDER_ID` указано пустое значение, рекомендация выполняется. Если `SG_ID` не пустое, перейдите к пункту «Инструкции и решения по выполнению».

{% endlist %}

**Инструкции и решения по выполнению:**

[Удалите](../../../cli/cli-ref/vpc/cli-ref/security-group/index.md) опасное правило в каждой группе безопасности или укажите доверенные IP-адреса.

#### 2.5 Включена защита от DDoS атак {#ddos-protection}

В {{ yandex-cloud }} существует базовая и расширенная защита от DDoS атак, а также защита на прикладном уровне с помощью сервиса {{ sws-full-name }}. Необходимо убедиться, что у вас используется как минимум базовая защита.

* [{{ sws-full-name }}](../../../smartwebsecurity/quickstart.md) — сервис для защиты от [DDoS-атак](../../../glossary/ddos.md) и ботов на прикладном уровне L7 [сетевой модели OSI](https://ru.wikipedia.org/wiki/Сетевая_модель_OSI). {{ sws-name }} [подключается](../../../smartwebsecurity/quickstart.md) к {{ alb-full-name }}. Функциональность сервиса сводится к проверке HTTP-запросов к защищаемому ресурсу на соответствие [правилам](../../../smartwebsecurity/concepts/rules.md), заданным в [профиле безопасности](../../../smartwebsecurity/concepts/profiles.md). В зависимости от результатов проверки запросы пропускаются на защищаемый ресурс, блокируются или отправляются в сервис [{{ captcha-full-name }}](../../../smartcaptcha/index.yaml) для дополнительной верификации.
* [{{ ddos-protection-full-name }}](../../../vpc/ddos-protection/index.md) — это компонент сервиса {{ vpc-name }} для защиты облачных ресурсов от DDoS-атак. {{ ddos-protection-name }} предоставляется в партнерстве с Curator. Вы можете включать ее самостоятельно на внешний [IP-адрес](../../../vpc/concepts/address.md) через инструменты управления облаком. Работает до L4 уровня модели OSI.
* [Расширенная](/services/ddos-protection) защита от DDoS-атак — работает на 3, 4 и 7 уровнях модели OSI. Вы также можете отслеживать показатели нагрузки, параметры атак и подключить Solidwall WAF в личном кабинете Curator. Чтобы включить расширенную защиту, обратитесь к вашему менеджеру или в техническую поддержку.

{% list tabs group=instructions %}

- Проверка в консоли управления {#console}

  * Чтобы убедиться, что у вас используется защита от DDoS атак на прикладном уровне:

      1. В [консоли управления]({{ link-console-main }}) выберите [каталог](../../../resource-manager/concepts/resources-hierarchy.md#folder), в котором вы хотите проверить статус {{ sws-name }}.
      1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_smartwebsecurity }}**.
      1. Убедитесь, что у вас есть созданные профили безопасности.
      1. Если профили безопасности есть, рекомендация выполняется. Если нет, перейдите к пункту «Инструкции и решения по выполнению».

  * Чтобы убедиться, что у вас используется базовая защита от DDoS атак:

      1. В [консоли управления]({{ link-console-main }}) откройте все созданные сети.
      1. Перейдите в раздел **IP-адреса**.
      1. Если у всех публичных адресов в столбце **Защита от DDoS-атак** установлено значение **Включена**, рекомендация выполняется. Если нет, перейдите к пункту «Инструкции и решения по выполнению».

- Ручная проверка {#manual}

  Чтобы убедиться, что у вас подключена расширенная защита от DDoS-атак, обратитесь к вашему персональному менеджеру. 

- Проверка через CLI {#cli}

  * Чтобы убедиться, что у вас используется защита от DDoS атак на прикладном уровне, выполните команду:

      ```bash
      yc smartwebsecurity security-profile list
      ```

      Если команда вернет информацию об имеющихся профилях безопасности, рекомендация выполняется. В противном случае перейдите к п. «Инструкции и решения по выполнению».

  * Чтобы убедиться, что у вас используется базовая защита от DDoS атак:

      1. Посмотрите доступные вам организации и зафиксируйте необходимый `ID`:

           ```bash
           yc organization-manager organization list
           ```

      1. Выполните команду для поиска IP-адресов без защиты от DDoS:

           ```bash
           export ORG_ID=<ID организации>
           for CLOUD_ID in $(yc resource-manager cloud list --organization-id=${ORG_ID} --format=json | jq -r '.[].id');
           do for FOLDER_ID in $(yc resource-manager folder list --cloud-id=$CLOUD_ID --format=json | jq -r '.[].id'); \
           do echo "Address_ID: " && yc vpc address list --folder-id=$FOLDER_ID \
           --format=json | jq -r '.[] | select(.external_ipv4_address.requirements.ddos_protection_provider=="qrator" | not)' | jq -r '.id' \
           && echo "FOLDER_ID: " $FOLDER_ID && echo "-----"
           done;
           done
           ```

      1. Если `Address_ID` напротив `FOLDER_ID` указано пустое значение, рекомендация выполняется. В противном случае перейдите к пункту «Инструкции и решения по выполнению».

{% endlist %}

**Инструкции и решения по выполнению:**

* [Инструкция по созданию профиля безопасности {{ sws-name }}](../../../smartwebsecurity/operations/profile-create.md).
* Вебинар [Защита от DDoS в {{ yandex-cloud }}](https://youtu.be/KWGbLQTth5U).
* Все [материалы](../../../vpc/ddos-protection/index.md) по защите от DDoS в {{ yandex-cloud }}.

#### 2.6 Используется защищенный удаленный доступ {#secure-access}

Чтобы обеспечить удаленное подключение администраторов к облачным ресурсам, используйте одно из следующих решений:
* Site-to-site VPN между удаленной площадкой (например, вашим офисом) и облаком. В качестве шлюза для удаленного доступа используйте ВМ с функцией site-to-site VPN на основе [образа]({{ link-cloud-marketplace }}?categories=network) из {{ marketplace-name }}.

  **Варианты настройки**:
  * [Создание туннеля IPSec VPN с использованием демона strongSwan](../../../tutorials/routing/ipsec/index.md).
  * [Создание site-to-site VPN-соединения с {{ yandex-cloud }} с помощью {{ TF }}](https://github.com/yandex-cloud-examples/yc-site-to-site-vpn-with-ipsec-strongswan).
  * Client VPN между удаленными устройствами и {{ yandex-cloud }}. В качестве шлюза для удаленного доступа используйте ВМ с функцией client VPN на основе [образа]({{ link-cloud-marketplace }}?categories=network) из {{ marketplace-name }}.

  См. инструкцию в разделе [Создание VPN-соединения с помощью OpenVPN](../../../tutorials/routing/openvpn.md). Возможно так же использование сертифицированных СКЗИ.
* Приватное выделенное соединение между удаленной площадкой и {{ yandex-cloud }} c помощью сервиса {{ interconnect-name }}.

Для доступа в инфраструктуру по управляющим протоколам (например, SSH, RDP) рекомендуется создать бастионную ВМ. Для этого можно использовать бесплатное решение [Teleport](https://goteleport.com/). Доступ к бастионной ВМ или VPN-шлюзу из интернета должен быть ограничен.

Для дополнительного контроля действий администраторов рекомендуется использовать решения PAM (Privileged Access Management) с записью сессии администратора (например, Teleport). Для доступа по SSH и VPN рекомендуется отказаться от паролей и вместо этого использовать открытые ключи, X.509-сертификаты и SSH-сертификаты. При настройке SSH для ВМ рекомендуется использовать SSH-сертификаты, в том числе и для хостовой части SSH.

Для доступа к веб-сервисам, развернутым в облаке, рекомендуется использовать TLS версий 1.2 и выше.

{% list tabs group=instructions %}

- Проверка в консоли управления {#console}

  1. Откройте консоль {{ yandex-cloud }} в вашем браузере.
  1. Откройте все созданные сети.
  1. Перейдите в раздел **Таблицы маршрутизации**.
  1. Если найдены маршруты в приватные сети удаленных площадок, которые направлены через ВМ с VPN шлюзом, рекомендация выполняется.
  1. Проверьте ВМ в каждом облаке на наличие VPN-шлюзов. Также проверьте у назначенных им групп безопасности открытые порты для VPN.

- Ручная проверка {#manual}

  Обратитесь к вашему персональному менеджеру и уточните, подключен ли у вас сервис {{ interconnect-name }}. Если подключен, проверьте, выполняется ли удаленный доступ.

{% endlist %}

#### 2.7 Для обеспечения удаленного доступа сотрудники используют {{ cloud-desktop-full-name }} {#use-cloud-desktop}

{{ cloud-desktop-full-name }} — сервис для управления виртуальной инфраструктурой [рабочих столов](../../../cloud-desktop/concepts/desktops-and-groups.md).

С помощью сервиса вы можете:

* быстро создавать виртуальные рабочие места для новых сотрудников;
* безопасно подключать сотрудников, работающих удаленно, к корпоративной сети;
* предоставлять сотрудникам возможность работать с любого современного устройства, имеющего доступ в интернет, в том числе личного ([BYOD](https://ru.wikipedia.org/wiki/Bring_your_own_device));
* управлять вычислительными ресурсами рабочих столов;
* удаленно администрировать рабочие столы;
* создавать группы рабочих столов с одинаковыми вычислительными ресурсами и [облачной сетью](../../../vpc/concepts/network.md).

{% list tabs group=instructions %}

- Проверка в консоли управления {#console}

  1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы хотите проверить наличие [рабочих столов](../../../cloud-desktop/concepts/desktops-and-groups.md).
  1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_cloud-desktop }}**.
  1. На панели слева выберите ![image](../../../_assets/console-icons/display.svg) **{{ ui-key.yacloud.vdi.label_desktops }}**.
  1. Если в списке есть хотя бы один созданный рабочий стол, то рекомендация выполняется. В противном случае, переходите к п. «Инструкции и решения по выполнению».

{% endlist %}

**Инструкции и решения по выполнению:**

1. [Создайте группу рабочих столов](../../../cloud-desktop/operations/desktop-groups/create.md).
1. Если у вас есть специфичные требования к настройке операционной системы, вы можете использовать свой собственный образ ОС, воспользовавшись инструкцией [{#T}](../../../cloud-desktop/operations/images/create-from-compute-linux.md), или [создать](../../../cloud-desktop/operations/images/create-from-desktop.md) образ на основе существующего рабочего стола и переиспользовать его для группы.
1. После создания рабочей группы администратор может [создать](../../../cloud-desktop/operations/desktops/create.md) нужное количество рабочих столов и самостоятельно назначить на них пользователей. Либо пользователи, входящие в группу рабочих столов, могут воспользоваться [витриной пользовательских рабочих столов](../../../cloud-desktop/concepts/showcase.md) и получить рабочий стол самостоятельно.

#### 2.8 Для удаленного доступа к {{ cloud-desktop-name }} используется Безопасный Яндекс Браузер {#use-yandex-browser}

Сотрудникам, работающим удаленно через {{ cloud-desktop-name }}, для доступа к корпоративным ресурсам рекомендуется использовать [Безопасный Яндекс Браузер](https://browser.yandex.ru/corp). Это требование направлено на обеспечение безопасности данных, защиту от [фишинга](https://ru.wikipedia.org/wiki/Фишинг), вредоносных сайтов и утечек информации. Браузер предоставляет встроенные инструменты для шифрования трафика, блокировки опасных ресурсов и интеграции с корпоративными системами аутентификации.

#### 2.9 Исходящий доступ в интернет контролируется {#outgoing-access}

Возможные варианты организации исходящего доступа в интернет:
* [Публичный IP-адрес](../../../vpc/concepts/address.md#public-addresses). Адрес назначается ВМ по принципу one-to-one NAT.
* [Egress NAT (NAT-шлюз)](../../../vpc/operations/create-nat-gateway.md). Включает доступ в интернет для подсети через общий пул публичных адресов {{ yandex-cloud }}. Не рекомендуется использовать Egress NAT для критичных взаимодействий, так как IP-адрес NAT-шлюза может использоваться несколькими клиентами одновременно. Следует учитывать эту особенность при моделировании угроз для инфраструктуры.
* [NAT-инстанс](../../../tutorials/routing/nat-instance/index.md). Функцию NAT выполняет отдельная ВМ. Для создания такой ВМ можно использовать образ [NAT-инстанс]({{ link-cloud-marketplace }}/products/yc/nat-instance-ubuntu-18-04-lts) из {{ marketplace-name }}.

**Сравнение способов доступа в интернет**:

#|
|| Публичный IP-адрес | Egress NAT | NAT-инстанс ||
|| **Плюсы:** | **Плюсы:** | **Плюсы:** ||
||
* Не требует настройки
* Выделенный адрес для каждой ВМ
|
* Не требует настройки
* Работает только на исходящих соединениях
|
* Возможность фильтровать трафик на NAT-инстансе
* Возможность использовать собственный файрвол
* Экономия IP-адресов ||
|| **Минусы:** | **Минусы:** | **Минусы:** ||
||
* Выставлять ВМ напрямую в интернет может быть небезопасно
* Стоимость резервирования каждого адреса |
* Общий пул IP-адресов
* Функция на стадии [Preview](../../../overview/concepts/launch-stages.md), поэтому не рекомендуется для продуктовых сред |
* Требуется настройка
* Стоимость использования ВМ (vCPU, RAM, диска) ||
|#

Вне зависимости от выбранного варианта организации исходящего доступа в интернет, ограничивайте трафик с помощью одного из механизмов, описанных выше. Для построения защищенной системы необходимо использовать статические IP-адреса, так как их можно внести в список исключений файрвола принимающей стороны.

{% list tabs group=instructions %}

- Проверка в консоли управления {#console}

  1. Откройте консоль {{ yandex-cloud }} в вашем браузере.
  1. Перейдите в нужный каталог.
  1. Перейдите в раздел **IP-адреса**.
  1. Если у всех публичных адресов в столбце **Защита от DDoS-атак** установлено значение **Включена**, рекомендация выполняется. В противном случае перейдите к пункту «Инструкции и решения по выполнению».

- Проверка через CLI {#cli}

  1. Посмотрите доступные вам организации и зафиксируйте необходимый `ID`:

     ```bash
     yc organization-manager organization list
     ```

  1. Выполните команду для поиска всех ВМ с публичными адресами:

     ```bash
     export ORG_ID=<ID организации>
     for CLOUD_ID in $(yc resource-manager cloud list --organization-id=${ORG_ID} --format=json | jq -r '.[].id');
     do for FOLDER_ID in $(yc resource-manager folder list --cloud-id=$CLOUD_ID --format=json | jq -r '.[].id');
     do echo "VM_ID: " && yc compute instance list --folder-id=$FOLDER_ID --format=json | jq -r '.[] | select(.network_interfaces[].primary_v4_address.one_to_one_nat.address)' | jq -r '.id' \
     && echo "FOLDER_ID: " $FOLDER_ID && echo "-----"
     done;
     done
     ```

  1. Если `VM_ID` напротив `FOLDER_ID` указано пустое значение, рекомендация выполняется. В противном случае перейдите к пункту «Инструкции и решения по выполнению».
  1. Выполните команду для поиска наличия Egress NAT (NAT-шлюз):

     ```bash
     export ORG_ID=<ID организации>
     for CLOUD_ID in $(yc resource-manager cloud list --organization-id=${ORG_ID} --format=json | jq -r '.[].id');
     do for FOLDER_ID in $(yc resource-manager folder list --cloud-id=$CLOUD_ID --format=json | jq -r '.[].id'); \
     do echo "NAT_GW: " && yc vpc gateway list --folder-id=$FOLDER_ID --format=json | jq -r '.[] | select(.id)' | jq -r '.id' && echo "FOLDER_ID: " $FOLDER_ID && echo "-----"
     done;
     done
     ```

  1. Если `NAT_GW` напротив `FOLDER_ID` указано пустое значение, рекомендация выполняется. В противном случае перейдите к пункту «Инструкции и решения по выполнению».
  1. Выполните команду для поиска наличия NAT-инстанса:

     ```bash
     export ORG_ID=<ID организации>
     for CLOUD_ID in $(yc resource-manager cloud list --organization-id=${ORG_ID} --format=json | jq -r '.[].id');
     do for FOLDER_ID in $(yc resource-manager folder list --cloud-id=$CLOUD_ID --format=json | jq -r '.[].id');
     do for DISK_ID in $(yc compute disk list --folder-id=$FOLDER_ID --format=json | jq -r '.[].id'); do yc compute disk get --id=$DISK_ID --format=json | jq -r '. | select(.product_ids[0]=="fd8v7ru46kt3s4o5f0uo")' | jq -r '.id'
     done;
     done;
     done
     ```

  1. Если результатом является пустая строка, рекомендация выполняется. Если видите `ID` NAT-инстанса, перейдите к пункту «Инструкции и решения по выполнению».

{% endlist %}

**Инструкции и решения по выполнению:**
* В случае наличия публичных адресов на ВМ убедитесь, что они необходимы. В противном случае удалите внешний IP-адрес в настройках ВМ.
* В случае наличия NAT-Gateway убедитесь, что он необходим. В противном случае удалите его.
* В случае наличия NAT-инстанс убедитесь, что он необходим. В противном случае удалите его.

#### 2.10 Запросы DNS не передаются в сторонние рекурсивные резолверы {#recursive-resolvers}

Для повышения отказоустойчивости часть трафика может передаваться в сторонние рекурсивные резолверы. Если необходимо избежать этого, обратитесь в [службу технической поддержки](../../../support/overview.md).
