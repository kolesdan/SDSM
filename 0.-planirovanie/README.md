# 0. Планирование

Это первая статья из серии «Сети для самых маленьких». Мы с Максимом aka Gluck долго думали с чего начать: маршрутизация, VLAN'ы, настройка оборудования.  
В итоге решили начать с вещи фундаментальной и, можно сказать, самой важной: планирование. Поскольку цикл рассчитан на совсем новичков, то и пройдём весь путь от начала до конца.


Предполагается, что вы, как минимум, читали о эталонной модели [OSI](http://ru.wikipedia.org/wiki/Сетевая_модель_OSI) \(то же на [англ.](http://en.wikipedia.org/wiki/OSI_model)\), о стеке протоколов [TCP/IP](http://ru.wikipedia.org/wiki/TCP/IP) \([англ.](http://en.wikipedia.org/wiki/TCP/IP_model)\), знаете о типах существующих [VLAN’ов](http://xgu.ru/wiki/VLAN) \(эту статью я настоятельно рекомендую к прочтению\), о наиболее популярном сейчас [port-based VLAN](http://en.wikipedia.org/wiki/IEEE_802.1Q) и о [IP адресах](http://xgu.ru/wiki/IP-адрес) \([более подробно](http://en.wikipedia.org/wiki/IP_address)\). Мы понимаем, что для новичков «OSI» и «TCP/IP» — это страшные слова. Но не переживайте, не для того, чтобы запугать вас, мы их используем. Это то, с чем вам придётся встречаться каждый день, поэтому в течение этого цикла мы постараемся раскрыть их смысл и отношение к реальности.

Начнём с постановки задачи. Есть некая фирма, занимающаяся, допустим, производством лифтов, идущих только вверх, и потому называется ООО «Лифт ми ап». Расположены они в старом здании на Арбате, и сгнившие провода, воткнутые в пожжёные и прожжёные коммутаторы времён 10Base-T не ожидают подключения новых серверов по гигабитным карточкам. Итак, у них катастрофическая потребность в сетевой инфраструктуре и денег куры не клюют, что даёт вам возможность безграничного выбора. Это чудесный сон любого инженера. А вы вчера выдержали собеседование, и в сложной борьбе по праву получили должность сетевого администратора. И теперь вы в ней первый и единственный в своём роде. Поздравляем! Что дальше?

Следует несколько конкретизировать ситуацию:

1. В данный момент у компании есть два офиса: 200 квадратов на Арбате под рабочие места и серверную. Там представлены несколько провайдеров. Другой на Рублёвке.
2. Есть четыре группы пользователей: бухгалтерия \(**Б**\), финансово-экономический отдел \(**ФЭО**\), производственно-технический отдел \(**ПТО**\), другие пользователи \(**Д**\). А так же есть сервера ©, которые вынесены в отдельную группу. Все группы разграничены и не имеют прямого доступа друг к другу.
3. Пользователи групп **С**, **Б** и **ФЭО **будут только в офисе на Арбате, **ПТО **и **Д** будут в обоих офисах.

Прикинув количество пользователей, необходимые интерфейсы, каналы связи, вы готовите схему сети и IP-план.

При проектировании сети следует стараться придерживаться [иерархической модели сети](http://en.wikipedia.org/wiki/Hierarchical_internetworking_model), которая имеет много достоинств по сравнению с “плоской сетью”:

* упрощается понимание организации сети
* модель подразумевает модульность, что означает простоту наращивания мощностей именно там, где необходимо
* легче найти и изолировать проблему
* повышенная отказоустойчивость за счет дублирования устройств и/или соединений
* распределение функций по обеспечению работоспособности сети по различным устройствам.

Согласно этой модели, сеть разбивается на три логических уровня: 

* **ядро сети** \(_Core layer_: высокопроизводительные устройства, главное назначение — быстрый транспорт\)
* **уровень распространения** \(_Distribution layer_: обеспечивает применение политик безопасности, QoS, агрегацию и маршрутизацию в VLAN, определяет широковещательные домены\) 
* **уровень доступа** \(Access-layer: как правило, L2 свичи, назначение: подключение конечных устройств, маркирование трафика для QoS, защита от колец в сети \(STP\) и широковещательных штормов, обеспечение питания для PoE устройств\).

В таких масштабах, как наш, роль каждого устройства размывается, однако логически разделить сеть можно. Составим приблизительную схему:

![Схема сети](http://img-fotki.yandex.ru/get/4/83739833.f/0_7c096_e09ecad8_XL.jpg)

На представленной схеме ядром \(_Core_\) будет маршрутизатор 2811, коммутатор 2960 отнесём к уровню распространения \(_Distribution_\), поскольку на нём агрегируются все VLAN в общий транк. Коммутаторы 2950 будут устройствами доступа \(_Access_\). К ним будут подключаться конечные пользователи, офисная техника, сервера.

Именовать устройства будем следующим образом: 

* сокращённое название города \(msk\) 
* географическое расположение \(улица, здание\) \(arbat\) 
* роль устройства в сети + порядковый номер.

Соответственно их ролям и месту расположения выбираем **hostname**:

| Оборудование | Hostname |
| :--- | :--- |
| Маршрутизатор 2811 | msk-arbat-gw1 \(gw=GateWay=шлюз\) |
| Коммутатор 2960 | msk-arbat-dsw1 \(dsw=Distribution switch\) |
| Коммутаторы 2950 | msk-arbat-aswN, msk-rubl-asw1 \(asw=Access switch\) |
