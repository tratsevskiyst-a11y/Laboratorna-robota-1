<!--
Бланк звіту до лабораторної роботи «DNS».
Скопіювати цей файл у власний публічний репозиторій звіту як README.md і заповнити.
Коментарі такого вигляду в GitHub не відображаються; видаляти їх не обов'язково.
Вивід команд вставляти текстом у блоки коду, а не знімками екрана.
Порожні поля не допускаються: якщо щось не вдалося, записати, що саме і чому.
-->

# Лабораторна робота. DNS

| Поле | Значення |
|---|---|
| ПІБ |Трацевський Станіслав Олегович |
| Група |2.01 |
| Ідентифікатор (`sNN`) |cs-201-24 |
| Дата виконання |03.11.2026 |
| Середовище | GitHub Codespaces |
| Версія `dig` | DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu |

## Параметри варіанта

<!-- Переписати з картки stand/cards/sNN.md. -->

| Параметр | Значення |
|---|---|
| `$TTL`, с |75 с |
| TTL запису SOA, с |120 с |
| Поле `minimum` SOA, с |90 с |
| Псевдонім (CNAME на `www`) |app |
| Другий хост і його TTL, с |next (TTL: 50 с) |
| Пріоритет MX |15 |
| Токен TXT |lab-token=76375453 |
| Ім'я для завдання 5 |new-cnxjc.cs-201-24.lab |

## Завдання 1. Складові URI

### 1.1. Розбір

<!-- Відсутню складову позначати «—». Порт за замовчуванням записувати з позначкою «(за замовч.)». -->

| № | scheme | userinfo | host | port | path | query | fragment |
|---|---|---|---|---|---|---|---|
| 1 |http |- |www.cs-201-24.lab |80 |/users/items |page=23&limit=asc |section-2 |
| 2 |http |- |app.cs-201-24.lab |80 |/items/ |page=%D0%BA%D0%B8%D1%97%D0%B2 |- |
| 3 |http |user:pass |next.cs-201-24.lab |80 |/users |- |section-2 |
| 4 |https |- |www.cs-201-24.lab |8443 |/users/../items |- |- |
| 5 |mailto |- |- |- |admin@cs-201-24.lab |subject=users |- |

| № | Ім'я, що запитується в DNS | Заголовок `Host` | Ціль запиту (request target) |
|---|---|---|---|
| 1 |www.cs-201-24.lab |www.cs-201-24.lab |/users/items?page=23&limit=asc |
| 2 |app.cs-201-24.lab |app.cs-201-24.lab:80 |/items/?page=%D0%BA%D0%B8%D1%97%D0%B2 |
| 3 |next.cs-201-24.lab |next.cs-201-24.lab |/users |
| 4 |www.cs-201-24.lab |www.cs-201-24.lab:8443 |/users/../items |
| 5 |- |- |- |

Декодоване значення параметра URI 2:Київ

Чому URI 5 не має складової host, хоча містить символ `@`:Схема mailto: створена для електронної пошти, а не для відкриття вебсайтів, тому їй не потрібне ім'я сервера (host). Через це весь текст після mailto: вважається просто шляхом, де символ @ ділить поштову адресу на ім'я користувача та домен.

### 1.2. Перевірка (після завдання 3)

<!-- Для URI 1–3: рядки запиту (починаються з ">") і тіло відповіді сервера. -->

```text
root@ns-cs-201-24:/# curl -v 'http://www.cs-201-24.lab/users/items?page=23&limit=asc#section-2'
* Host www.cs-201-24.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to www.cs-201-24.lab (10.53.0.80) port 80
> GET /users/items?page=23&limit=asc HTTP/1.1
> Host: www.cs-201-24.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Wed, 04 Nov 2026 00:18:15 GMT
< Content-Type: text/plain
< Content-Length: 144
< Connection: keep-alive
< 
server: web-a
Host: www.cs-201-24.lab
request URI: /users/items?page=23&limit=asc
path: /users/items
query: page=23&limit=asc
client: 10.53.1.1
* Connection #0 to host www.cs-201-24.lab left intact
root@ns-cs-201-24:/# curl -v 'http://app.cs-201-24.lab:80/items/?page=%D0%BA%D0%B8%D1%97%D0%B2'
* Host app.cs-201-24.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to app.cs-201-24.lab (10.53.0.80) port 80
> GET /items/?page=%D0%BA%D0%B8%D1%97%D0%B2 HTTP/1.1
> Host: app.cs-201-24.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Wed, 04 Nov 2026 00:18:55 GMT
< Content-Type: text/plain
< Content-Length: 158
< Connection: keep-alive
< 
server: web-a
Host: app.cs-201-24.lab
request URI: /items/?page=%D0%BA%D0%B8%D1%97%D0%B2
path: /items/
query: page=%D0%BA%D0%B8%D1%97%D0%B2
client: 10.53.1.1
* Connection #0 to host app.cs-201-24.lab left intact
root@ns-cs-201-24:/# curl -v 'http://user:pass@next.cs-201-24.lab/users#section-2'
* Host next.cs-201-24.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.81
*   Trying 10.53.0.81:80...
* Connected to next.cs-201-24.lab (10.53.0.81) port 80
* Server auth using Basic with user 'user'
> GET /users HTTP/1.1
> Host: next.cs-201-24.lab
> Authorization: Basic dXNlcjpwYXNz
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Wed, 04 Nov 2026 00:19:04 GMT
< Content-Type: text/plain
< Content-Length: 98
< Connection: keep-alive
< 
server: web-b
Host: next.cs-201-24.lab
request URI: /users
path: /users
query: 
client: 10.53.1.1
* Connection #0 to host next.cs-201-24.lab left intact
```

Значення заголовка `Authorization` і результат декодування:

```text
Authorization: Basic dXNlcjpwYXNz
Результат декодування: user:pass
```

URI 4: етап, на якому запит не вдається (пошук імені / TCP / TLS), і рядки виводу, що це підтверджують:

```text
* Trying 10.53.0.80:8443...
* connect to 10.53.0.80 port 8443 from 10.53.1.1 port 47136 failed: Connection refused
* Failed to connect to www.cs-201-24.lab port 8443 after 0 ms: Couldn't connect to server
curl: (7) Failed to connect to www.cs-201-24.lab port 8443 after 0 ms: Couldn't connect to server
```[cite: 16]
```

| Запит | `request URI`, отриманий сервером | `path`, отриманий сервером |
|---|---|---|
| звичайний |/users/items?page=23&limit=asc|/users/items |
| `--path-as-is` |/users/../items |/users/../items |

Розбіжності між URI та тим, що отримав сервер, і їх пояснення:
Коли відправляється запит через curl, програма може автоматично змінювати шлях перед тим, як відправити його в мережу (наприклад, прибирати зайві перехода типу /../, якщо не вказано спеціальний прапорець --path-as-is). Крім того, все, що вказано після символу решітки, вважається внутрішнім фрагментом або якірем для браузера, тому клієнт взагалі не передає цю частину на сервер. Через це сервер отримує лише ту частину шляху та параметрів, яка реально долетіла по мережі, іноді у дещо зміненому чи нормалізованому вигляді.
## Завдання 2. Проходження ієрархії вручну

```text
root@ns-cs-201-24:/# dig @10.53.0.2 ns.cs-201-24.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.0.2 ns.cs-201-24.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 27652
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 8a04d47369f2d296010000006afc58d1be36c2acdd95c109 (good)
;; QUESTION SECTION:
;ns.cs-201-24.lab.              IN      A

;; AUTHORITY SECTION:
lab.                    172800  IN      NS      ns1.nic.lab.

;; ADDITIONAL SECTION:
ns1.nic.lab.            172800  IN      A       10.53.0.3

;; Query time: 81 msec
;; SERVER: 10.53.0.2#53(10.53.0.2) (UDP)
;; WHEN: Tue Nov 17 13:38:25 UTC 2026
;; MSG SIZE  rcvd: 117
root@ns-cs-201-24:/# dig @10.53.0.3 ns.cs-201-24.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.0.3 ns.cs-201-24.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52699
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: d27913b6ccb53974010000006afc58ff2df0b744dbb841b9 (good)
;; QUESTION SECTION:
;ns.cs-201-24.lab.              IN      A

;; AUTHORITY SECTION:
cs-201-24.lab.          3600    IN      NS      ns.cs-201-24.lab.

;; ADDITIONAL SECTION:
ns.cs-201-24.lab.       3600    IN      A       10.53.1.1

;; Query time: 80 msec
;; SERVER: 10.53.0.3#53(10.53.0.3) (UDP)
;; WHEN: Tue Nov 17 13:39:11 UTC 2026
;; MSG SIZE  rcvd: 103
root@ns-cs-201-24:/# dig @10.53.1.1 ns.cs-201-24.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 ns.cs-201-24.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 47885
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 21cd5f7c617d3ada010000006afc592a6d5b23da1138746f (good)
;; QUESTION SECTION:
;ns.cs-201-24.lab.              IN      A

;; ANSWER SECTION:
ns.cs-201-24.lab.       75      IN      A       10.53.1.1

;; AUTHORITY SECTION:
cs-201-24.lab.          75      IN      NS      ns.cs-201-24.lab.

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Tue Nov 17 13:39:54 UTC 2026
;; MSG SIZE  rcvd: 103
```

| Сервер | status | flags | ANSWER | AUTHORITY | ADDITIONAL |
|---|---|---|---|---|---|
| 10.53.0.2 |NOERROR |qr |0 |1 |2 |
| 10.53.0.3 |NOERROR |qr |0 |1 |2 |
| 10.53.1.1 |NOERROR |qr |1 |1 |1 |

Вивід `dig ns.sNN.lab A +trace`:

```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> ns.cs-201-24.lab A +trace
;; global options: +cmd
.                       82092   IN      NS      a.root-servers.net.
;; Received 59 bytes from 10.53.0.53#53(10.53.0.53) in 0 ms

lab.                    172800  IN      NS      ns1.nic.lab.
;; Received 117 bytes from 10.53.0.2#53(a.root-servers.net) in 81 ms

cs-201-24.lab.          3600    IN      NS      ns.cs-201-24.lab.
;; Received 103 bytes from 10.53.0.3#53(ns1.nic.lab) in 42 ms

ns.cs-201-24.lab.       75      IN      A       10.53.1.1
cs-201-24.lab.          75      IN      NS      ns.cs-201-24.lab.
;; Received 103 bytes from 10.53.1.1#53(ns.cs-201-24.lab) in 0 ms
```

| Запит | TTL запису NS | Прапорець `aa` | Секція, у якій міститься NS |
|---|---|---|---|
| `dig @10.53.0.3 sNN.lab NS +norec` |3600 |Відсутній |AUTHORITY SECTION |
| `dig @10.53.1.1 sNN.lab NS +norec` |75 |Присутній |ANSWER SECTION |

1. Які відповіді є відсиланнями і за якими ознаками це визначено?
Відсиланнями є відповіді від кореневого та TLD-серверів, оскільки їхня секція ANSWER порожня, а перенаправлення на наступний сервер міститься в секції AUTHORITY.
2. Для чого у відповіді TLD присутній запис у секції ADDITIONAL і що сталося б, якби його не було?
Запис у секції ADDITIONAL містить IP-адресу авторитативного сервера , без якої резолвер не зміг би знайти шлях до нього через замкнене коло залежностей.
3. Яка з двох копій запису NS є авторитативною і чому?
Авторитативною є копія запису NS від власного сервера, оскільки він є безпосереднім власником цієї зони, повертає результат у секції ANSWER та містить прапорець aa.
## Завдання 3. Заповнення власної зони

Вивід `zone-reload`:

```text
root@ns-cs-201-24:/# zone-reload
zone cs-201-24.lab/IN: loaded serial 2026110402
OK
zone reload queued
```

<!-- Шість запитів до 10.53.1.1 з +norec, по одному на кожен рядок таблиці картки. -->

| Запис | status | Прапорець `aa` | Значення в ANSWER | TTL | Відповідає картці (так/ні) |
|---|---|---|---|---|---|
| `www` A |NOERROR |так |10.53.0.80 |75 |так |
| псевдонім CNAME |NOERROR |так |www.cs-201-24.lab. |75 |так |
| `mail` A |NOERROR |так |10.53.0.80 |75 |так |
| `@` MX |NOERROR |так |15 mail.cs-201-24.lab. |75 |так |
| `@` TXT |NOERROR |так |"lab-token=76375453" |75 |так |
| другий хост A |NOERROR |так |10.53.0.81 |50 |так |

Порівняння відповідей резолвера та власного сервера:

```text
При запиті напряму до власного сервера без рекурсії (+norec) відповідь містить прапорець aa (авторитетна відповідь) і включає секцію AUTHORITY із записом NS та секцію ADDITIONAL із його IP-адресою. Під час запиту через ресолвер сервер додає прапорець rd і прагне надати кінцеву відповідь, тому для псевдоніма CNAME одразу розгортає його в підсумковий A-запис у секції ANSWER.
```

| Запит | flags | ANSWER (кількість, типи) | AUTHORITY (кількість, типи) | ADDITIONAL (кількість, типи) | TTL |
|---|---|---|---|---|---|
| псевдонім через resolver-a |qr aa rd |2 (CNAME, A) |0 |1 (OPT) |75 |
| псевдонім через 10.53.1.1 |qr aa |2 (CNAME, A) |1 (NS) |2 (OPT, A) |75 |
| MX через resolver-a |qr aa rd |1 (MX) |0 |2 (OPT, A) |75 |
| MX через 10.53.1.1 |qr aa |1 (MX) |1 (NS) |3 (OPT, A, A) |75 |

Пояснення відмінностей:
Коли запит надсилається напряму до сервера без рекурсії, сервер відповідає як прямий господар зони: ставить прапорець aa, підтверджує свої повноваження, додає в секцію AUTHORITY дані про сервер імен NS, а в секцію ADDITIONAL записує свій IP-адрес. При звичайному записі через ресолвер сервер прагне видати максимально готовий результат: якщо у запиті був псевдонім CNAME, він автоматично розгортає його до кінцевого A-запису і одразу вміщує всі дані в секцію ANSWER.
Повідомлення `zone-check` після тимчасового додавання `@ IN CNAME www`:

```text
dns_master_load: /lab/zone/cs-201-24.lab.zone:21: cs-201-24.lab: CNAME and other data
zone cs-201-24.lab/IN: loading from master file /lab/zone/cs-201-24.lab.zone failed: CNAME and other data
zone cs-201-24.lab/IN: not loaded due to errors.
```

1. Який TTL має кожен запис зони і звідки це значення взято?
Записи www, app, mail, MX та TXT мають TTL 75, оскільки це значення задано глобальною директивою $TTL 75 на початку файлу зони. Запис next має TTL 50, тому що для нього значення 50 було явно вказано безпосередньо в самому записі (next 50 IN A 10.53.0.81), що перевизначає глобальний $TTL.
2. Скільки записів містить секція ANSWER у відповіді на запит псевдоніма і чому?
Секція ANSWER містить 2 записи (CNAME та A). Це відбувається тому, що DNS-сервер при запиті псевдоніма app не лише повертає сам канонічний запис CNAME (app -> www), але й одразу розгортає його, додаючи пов'язаний запис A для www (10.53.0.80), щоб клієнту не доводилося робити повторний запит.
3. Чому запис CNAME неможливо розмістити для імені `@`?
За специфікацією DNS (RFC 1034 / RFC 1912), якщо ім'я має запис CNAME, воно не може мати жодних інших типів записів. Оскільки вершина зони обов'язково повинна містити записи SOA та NS для функціонування всієї зони, розміщення CNAME для @ створює неприпустимий синтаксичний конфлікт (CNAME and other data).
## Завдання 4. TTL і видимість змін

Два запити з інтервалом 10 с:

```text
root@ns-cs-201-24:/# dig www.cs-201-24.lab +noall +answer
www.cs-201-24.lab.      75      IN      A       10.53.0.80
root@ns-cs-201-24:/# dig www.cs-201-24.lab +noall +answer
www.cs-201-24.lab.      65      IN      A       10.53.0.80
```

**Прогноз** (записати до вимірювання). Формула часу появи нової адреси через залишок R, найбільша можлива затримка:
Формула: Час появи = Час zone-reload + R
Перевірка, що сервер до `zone-reload` повертає стару адресу:

```text
root@ns-cs-201-24:/# dig @10.53.1.1 www.cs-201-24.lab +norec +short
10.53.0.80
```

Залишок R і час (крок 4), час `zone-reload` (крок 5), перевірка на сервері (крок 6):

```text
root@ns-cs-201-24:/# date +%T; dig www.cs-201-24.lab +noall +answer
00:50:05
www.cs-201-24.lab.      75      IN      A       10.53.0.80
root@ns-cs-201-24:/# zone-reload; date +%T
zone cs-201-24.lab/IN: loaded serial 2026110303
OK
zone reload queued
00:50:22
root@ns-cs-201-24:/# dig @10.53.1.1 www.cs-201-24.lab +norec +short
10.53.0.81
```

Очікуваний час появи нової адреси (підставити R у формулу):
00:50:22 + 75 секунд=00:51:37
Вивід циклу спостереження (перші два рядки, два останні рядки зі старою адресою, рядок з новою адресою):

```text
00:51:33 10.53.0.80
00:51:35 10.53.0.80

00:53:28 10.53.0.80
00:53:30 10.53.0.80
00:53:32 10.53.0.81
```

Рядок `server:` з відповіді `curl`:
server: web-b
| Час зміни | Очікуваний час | Спостережений час | Виміряна затримка, с | Прогноз, с |
|---|---|---|---|---|
|00:50:22 |00:51:37 |00:53:32 |приблизно 190 с |75 с |

1. Чим пояснюється розбіжність між прогнозом і виміряним значенням?
Розбіжність між прогнозом і виміряним значенням пояснюється тим, що реальний час оновлення кешу залежить від інтервалів перевірки ресолвера та точного моменту надсилання запиту клієнтом після закінчення терміну дії попереднього закешованого запису.
2. Якою була б найбільша можлива затримка для цього запису і за яких умов?
Найбільша можлива затримка для цього запису дорівнювала б повному значенню TTL, тобто 75 секунд, і вона виникла б за умови, якби зміни до файлу зони внесли одразу після того, як ресолвер щойно закешував цей запис.
3. Як слід діяти, щоб перенести сервіс на нову адресу, якщо TTL запису становить 86 400 с?
Щоб перенести сервіс на нову адресу за великого TTL, слід заздалегідь, приблизно за добу до запланованої міграції, зменшити значення TTL у файлі зони до мінімуму, дочекатися закінчення терміну дії старого кешу, виконати перенесення на нову IP-адресу, а після успішного переходу за потреби знову збільшити TTL.
## Завдання 5. Негативне кешування

**Прогноз** (записати до вимірювання). Формула за RFC 2308 і розраховане значення:
RFC 2308: TTL_neg = min(TTL_SOA, SOA.MINIMUM) = min(120, 90) = 90 секунд
Перевірка, що сервер до `zone-reload` повертає `NXDOMAIN`:

```text
root@ns-cs-201-24:/# dig @10.53.1.1 new-cnxjc.cs-201-24.lab +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 new-cnxjc.cs-201-24.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 50154
;; flags: qr aa; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 3148466e2c71fa41010000006aea89e4a01f9cce6c015a21 (good)
;; QUESTION SECTION:
;new-cnxjc.cs-201-24.lab.       IN      A

;; AUTHORITY SECTION:
cs-201-24.lab.          90      IN      SOA     ns.cs-201-24.lab. admin.cs-201-24.lab. 2026110303 3600 600 86400 90

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Wed Nov 04 01:27:00 UTC 2026
;; MSG SIZE  rcvd: 125
```

Запит через resolver-a (крок 3), час `zone-reload` (крок 4), перевірка на сервері (крок 5):

```text
root@ns-cs-201-24:/# date +%T; dig new-cnxjc.cs-201-24.lab +noall +authority
01:28:31
cs-201-24.lab.          90      IN      SOA     ns.cs-201-24.lab. admin.cs-201-24.lab. 2026110303 3600 600 86400 90
root@ns-cs-201-24:/# zone-reload; date +%T
zone cs-201-24.lab/IN: loaded serial 2026110304
OK
zone reload queued
01:29:23
root@ns-cs-201-24:/# dig @10.53.1.1 new-cnxjc.cs-201-24.lab +norec +short
10.53.0.80
```

Вивід циклу спостереження (перші два рядки, останній рядок з `NXDOMAIN`, перший рядок з `NOERROR`):

```text
01:29:57 status: NXDOMAIN
01:29:59 status: NXDOMAIN
01:30:01 status: NXDOMAIN
01:30:03 status: NOERROR
```

| Негативну відповідь закешовано | Запис створено | Ім'я резолвиться через resolver-a | Виміряно, с | Прогноз, с |
|---|---|---|---|---|
|01:28:31 |01:29:23 |01:30:03 |92 с |90 с |

<!-- Якщо завдання повторювалося з іншим ім'ям (new2-…), зазначити це й причину. -->

1. Чому тривалість негативного кешування визначається записом SOA, а не записом, який запитували?
Оскільки запрашуване доменне ім'я фізично не існує у зоні, у нього немає власного запису та власного параметра TTL, тому ресолвер бере час негативного кешування із мінімального значення (або TTL) запису SOA власної зони.
2. Чи збігається TTL запису SOA в секції AUTHORITY з TTL запису SOA у файлі зони? Поясніть.
Ці значення збігаються, якщо запитувати інформацію напряму з авторитетного сервера, проте при зверненні через клієнтський резолвер час життя (TTL) починає зменшуватися з кожною секундою перебування відповіді в кеші ресолвера.
3. Скрипт розгортання перевіряє ім'я сервісу до створення запису, а потім одразу після. Що він побачить і як це виправити?
Скрипт побачить помилку про те, що ім'я не існує (NXDOMAIN), навіть після свого створення, оскільки перша перевірка закешувала негативну відповідь; щоб це виправити, перед перевіркою потрібно звертатися напряму до авторитетного сервера без кешування або заздалегідь зменшити значення мінімального TTL у записі SOA.
## Завдання 6. Затримка пошуку імені

**Прогноз** (записати до вимірювання). До яких серверів звернеться resolver-a:
А (нове ім'я у власній зоні): cs-201-24.
В (нове ім'я в зоні .lab): root, tld.
С (ім'я в неіснуючому TLD): root.
D (www.cs-201-24.lab після перезапуску):root, tld, cs-201-24.
| Ім'я | Перший запит | Повторний запит |
|---|---|---|
| A: нове ім'я у власній зоні |приблизно 1 мс|приблизно 0–1 мс |
| B: нове ім'я в зоні `lab.` |приблизно 40 мс|приблизно 0–1 мс |
| C: ім'я в неіснуючому TLD |приблизно 80 мс|приблизно 0–1 мс |
| D: `www.sNN.lab` після перезапуску резолвера |приблизно 400 мс|приблизно 0–1 мс |

Вивід вимірювань:

```text
root@ns-cs-201-24:/# A=y$RANDOM.cs-201-24.lab; dig $A | grep 'Query time'; dig $A | grep 'Query time'
;; Query time: 1 msec
;; Query time: 1 msec
root@ns-cs-201-24:/# A=y$RANDOM.cs-201-24.lab; dig $A | grep 'Query time'; dig $A | grep 'Query time'
;; Query time: 1 msec
;; Query time: 0 msec
root@ns-cs-201-24:/# B=x$RANDOM.lab; dig $B | grep 'Query time'; dig $B | grep 'Query time'
;; Query time: 41 msec
;; Query time: 0 msec
root@ns-cs-201-24:/# B=x$RANDOM.lab; dig $B | grep 'Query time'; dig $B | grep 'Query time'
;; Query time: 41 msec
;; Query time: 1 msec
root@ns-cs-201-24:/# C=x$RANDOM.zz$RANDOM; dig $C | grep 'Query time'; dig $C | grep 'Query time'
;; Query time: 81 msec
;; Query time: 1 msec
root@ns-cs-201-24:/# C=x$RANDOM.zz$RANDOM; dig $C | grep 'Query time'; dig $C | grep 'Query time'
;; Query time: 82 msec
;; Query time: 0 msec
root@ns-cs-201-24:/# dig www.cs-201-24.lab

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> www.cs-201-24.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 34114
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;www.cs-201-24.lab.             IN      A

;; ANSWER SECTION:
www.cs-201-24.lab.      75      IN      A       10.53.0.81

;; Query time: 402 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Wed Nov 04 02:04:59 UTC 2026
;; MSG SIZE  rcvd: 62

root@ns-cs-201-24:/# dig www.cs-201-24.lab

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> www.cs-201-24.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 55070
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;www.cs-201-24.lab.             IN      A

;; ANSWER SECTION:
www.cs-201-24.lab.      73      IN      A       10.53.0.81

;; Query time: 0 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Wed Nov 04 02:05:01 UTC 2026
;; MSG SIZE  rcvd: 62

```

| Ім'я | Перший запит, мс | Повторний, мс | Сервери, до яких звернувся резолвер (за журналами) |
|---|---|---|---|
| A |1 |0 |cs-201-24 |
| B |41 |0 |root, tld |
| C |82 |0 |root |
| D |402 |0 |root, tld, cs-201-24 |

Значення `time_namelookup` з `curl` після перезапуску резолвера:
0.404317 с
Фрагменти журналів запитів root, TLD і власного сервера для рядка D:

```text
04-Nov-2026 02:08:49.583 client @0x7cfecc001ee8 10.53.0.53#54895 (.): query: . IN NS -E(0)DC (10.53.0.2)
04-Nov-2026 02:08:49.745 client @0x7cfecc001ee8 10.53.0.53#29433 (www.cs-201-24.lab): query: www.cs-201-24.lab IN A -E(0)DC (10.53.0.2)
04-Nov-2026 02:08:49.866 client @0x72804406ac88 10.53.0.53#63114 (www.cs-201-24.lab): query: www.cs-201-24.lab IN A -E(0)DC (10.53.0.3)
04-Nov-2026 02:08:49.906 client @0x747d4c006018 10.53.0.53#64866 (www.cs-201-24.lab): query: www.cs-201-24.lab IN A -E(0)DC (10.53.1.1)
```

1. Яку затримку вносить кожен із серверів root, TLD і власний сервер? Покажіть, з яких рядків таблиці це випливає.
Власний сервер вносить мінімальну затримку близько 1 мс (рядок A). TLD-сервер додає ще близько 40 мс (різниця між рядком B та A). Кореневий сервер (root) додає близько 80 мс (рядок C). Загальний час повного рекурсивного запиту після перезапуску (рядок D) складається з суми затримок усіх рівнів ієрархії та мережевих витрат (приблизно 402 мс).
2. Чому в рядку A перший запит не проходить через root і TLD, хоча ім'я нове?
Оскільки цей домен належить до власної авторитетної зони (cs-201-24.lab), локальний ресолвер заздалегідь налаштований знати адресу авторитетного сервера для цієї зони (або є її частиною), тому йому не потрібно виконувати глобальний ітеративний пошук через кореневі сервери.
3. Вебсторінка завантажує ресурси з п'яти різних доменів. Як це впливає на час першого відкриття сторінки?
Якщо вебсторінка завантажує вміст одразу з п'яти різних сайтів, її відкриття помітно сповільнюється. Для кожної окремої адреси програмі доводиться спочатку проходити весь довгий шлях пошуку в інтернеті — опитувати кореневий сервер, потім сервер зони і лише після цього авторитетний сервер. Через те, що ці пошуки виконуються для кожного домену окремо, затримки додаються одна до одної, і користувач змушений довше чекати, поки сторінка почне завантажуватися.
## Завдання 7. Інцидент: зупинка власного сервера

**Прогноз** (записати до вимірювання). Що поверне кожен резолвер після зупинки сервера та спливання TTL і скільки часу займе відповідь:
resolver-a: поверне помилку SERVFAIL або тайм-аут, оскільки кеш застарів, а авторитетний сервер недоступний. Відповідь займе близько 3 секунд. resolver-b: поверне закешовану IP-адресу із застарілого кешу завдяки механізму Serve-Stale. Відповідь буде практично миттєвою (до 1–2 секунд).
Кешування запису в обох резолверах і час зупинки сервера:

```text
root@ns-cs-201-24:/# dig @10.53.0.53 www.cs-201-24.lab +noall +answer
www.cs-201-24.lab.      75      IN      A       10.53.0.81
root@ns-cs-201-24:/# dig @10.53.0.54 www.cs-201-24.lab +noall +answer
www.cs-201-24.lab.      75      IN      A       10.53.0.81
root@ns-cs-201-24:/# named-stop
root@ns-cs-201-24:/# sleep 90
```

Запити із заміром часу (крок 5):

```text
root@ns-cs-201-24:/# time dig @10.53.0.53 www.cs-201-24.lab +tries=1 +timeout=30
;; communications error to 10.53.0.53#53: timed out

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.0.53 www.cs-201-24.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; no servers could be reached

real    0m30.161s
user    0m0.004s
sys     0m0.009s
root@ns-cs-201-24:/# time dig @10.53.0.53 www.cs-201-24.lab +tries=1 +timeout=30

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.0.53 www.cs-201-24.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 31182
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;www.cs-201-24.lab.             IN      A

;; Query time: 0 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Tue Nov 17 14:20:26 UTC 2026
;; MSG SIZE  rcvd: 46


real    0m0.024s
user    0m0.008s
sys     0m0.004s
root@ns-cs-201-24:/# time dig @10.53.0.54 www.cs-201-24.lab +tries=1 +timeout=30

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.0.54 www.cs-201-24.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 41197
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;www.cs-201-24.lab.             IN      A

;; ANSWER SECTION:
www.cs-201-24.lab.      30      IN      A       10.53.0.81

;; Query time: 1800 msec
;; SERVER: 10.53.0.54#53(10.53.0.54) (UDP)
;; WHEN: Tue Nov 17 14:20:34 UTC 2026
;; MSG SIZE  rcvd: 62


real    0m1.823s
user    0m0.008s
sys     0m0.003s
```

| Запит | status або тайм-аут | ANSWER, TTL | `real` |
|---|---|---|---|
| resolver-a, перший |timed out |порожньо (0 записів) |0m30.161s|
| resolver-a, другий |SERVFAIL |порожньо (0 записів) |0m0.024s|
| resolver-b |NOERROR |www.cs-201-24.lab. 30 IN A 10.53.0.81 |0m1.823s |

Час `named-start` і вивід циклу спостереження до першої успішної відповіді (крок 6):

```text
root@ns-cs-201-24:/# named-start
root@ns-cs-201-24:/# while true; do echo "$(date +%T) $(dig @10.53.0.53 www.cs-201-24.lab +short +tries=1 +timeout=3)"; sleep 2; done
14:20:42 
14:20:44 
14:20:46 
14:20:48 
14:20:50 
14:20:52 
14:20:54 
14:20:56 
14:20:58 
14:21:01 10.53.0.81
```

Перевірка після завдання (`dig @10.53.1.1 sNN.lab SOA +norec +short`):

```text
root@ns-cs-201-24:/# dig @10.53.1.1 cs-201-24.lab SOA +norec +short
ns.cs-201-24.lab. admin.cs-201-24.lab. 2026110404 3600 600 86400 90
```

### Постмортем

1. Що сталося: Авторитетний сервер домену був зупинений за допомогою команди named-stop, через що перестав відповідати на зовнішні запити.
2. Вплив на клієнтів resolver-a і resolver-b: Звичайний resolver-a видавав тайм-аут і помилку, тоді як resolver-b завдяки функції Serve-Stale успішно віддавав застарілі дані з кешу.
3. Хронологія з виміряним часом: О 14:18:52 зупинено сервер; о 14:20:26 зафіксовано відмову resolver-a  зі статусом SERVFAIL, о 14:20:34 стабільну відповідь resolver-b зі статусом NOERROR, о 14:20:42 виконано named-start, 0 14:21:01 відновив успішне резолвлення імені
4. Чому кеш допоміг або не допоміг: Кеш допоміг на resolver-b, дозволивши видати застарілу інформацію під час збою, тоді як стандартний resolver-a без Serve-Stale не зміг відповісти клієнту.
5. Що слід змінити: Слід увімкнути підтримку Serve-Stale на всіх ресолверах для підвищення стійкості системи до падінь авторитетних серверів.

1. Який TTL мав запис у відповіді resolver-b і навіщо такий TTL?
Запис мав мінімальний або нульовий TTL, оскільки його реальний термін дії вже закінчився і ресолвер віддавав застарілі дані з кешу.
2. Порівняйте результат і тривалість двох запитів до resolver-a. Якщо вони різняться, сформулюйте гіпотезу щодо причини; якщо ні, поясніть, що робить резолвер протягом цього часу.
Обидва запити до resolver-a завершилися однаковими таймаутами, бо ресолвер усередині цього часу нескінченно намагався достукатися до вимкненого сервера.
3. Чому після `named-start` ім'я через resolver-a зарезолвилося не одразу?
Після named-start ім'я ресолвилося не одразу через невелику затримку на повторну ініціалізацію служби та встановлення мережевого з'єднання.
4. У якій ситуації serve-stale шкодить, а не допомагає? Пов'яжіть відповідь із завданням 4.
Serve-Stale шкодить тоді, коли IP-адреса сервісу критично змінилася (як у завданні 4), адже клієнт отримує застарілу стару адресу замість нової.
## Фінальний файл зони

<!-- Повний вміст stand/students/sNN/zone/sNN.lab.zone після завершення роботи. -->

```text
; Zone cs-201-24.lab  ^`^t starting version.
; After every change: increase the serial, then run  zone-reload
$TTL 75
@     120 IN SOA ns.cs-201-24.lab. admin.cs-201-24.lab. (
            2026110404 ; serial
            3600       ; refresh
            600        ; retry
            86400      ; expire
            90         ; minimum
        )
@     IN NS ns.cs-201-24.lab.
ns    IN A  10.53.1.1

; ---- add records from your variant card below this line ----
www     IN A     10.53.0.81
app     IN CNAME www
mail    IN A     10.53.0.80
@       IN MX 15 mail
@       IN TXT "lab-token=76375453"
next    50 IN A  10.53.0.81
new-cnxjc IN A 10.53.0.80
```

## Висновки

**Що виявилося несподіваним.** Результат, який не збігся з очікуванням, і рядок виводу, де це видно:
Несподіваним стало те, що після запуску сервера командою named-start резолвер резолвер-а запрацював не миттєво, а лише через 19 секунд. У виводі циклу спостереження з 14:20:42 до 14:20:58 з’являлися порожні рядки, і тільки о 14:21:01 повернулася адреса 10.53.0.81. Це сталося через роботу негативного кешу: після попередніх невдалих спроб резолвер запам'ятав, що сервер недоступний, і певний час просто не надсилав до нього нових запитів, чекаючи завершення внутрішнього таймера.
**Яке питання залишилося без відповіді.** Що саме не вдалося пояснити; до якого рядка виводу це стосується; що вже перевірено (RFC, документація, додатковий експеримент на стенді) і що з'ясувати не вдалося:
Залишилося невідомим, як довго резолвер-б здатний віддавати застарілий запис www.cs-201-24.lab. 30 IN A 10.53.0.81 із кешу, якщо авторитативний сервер буде вимкнений годинами або днями.
## Декларація використання ШІ

Використовувалися інструменти ШІ: <!-- так / ні -->

<!-- Якщо так, заповнити таблицю: один рядок на кожне звернення, що вплинуло на звіт. Промпт наводити дослівно. -->

| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 |Gemini |«розшир мій висновок десь до 120 слів» |Взято за основу розгорнутий аналіз затримки ресолвера та Serve-Stale, створений за допомогою штучного інтелекту. Текст самостійно вичитано, виправлено та узгоджено з реальними показниками, отриманими під час виконання лабораторної. |
