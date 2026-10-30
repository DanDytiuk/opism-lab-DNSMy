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
| ПІБ | Дитюк Даніїл |
| Група | ІПЗ 2.01 |
| Ідентифікатор (`sNN`) | se-201-04 |
| Дата виконання | 22.10.26 21:00 - 00:00 |
| Середовище | GitHub Codespaces |
| Версія `dig` | DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu |

## Параметри варіанта

# Варіант se-201-04 ()

**Доступ до терміналу:** http://<адреса стенду>:7001/  логін `se-201-04`
**Зона:** `se-201-04.lab`  **Авторитативний сервер:** `10.53.1.1`
**Резолвери:** `10.53.0.53` (resolver-a), `10.53.0.54` (resolver-b)

## Параметри зони (вже записані у стартовий файл)

| Параметр | Значення |
|---|---|
| `$TTL` (за замовчуванням для записів) | 120 с |
| TTL самого запису SOA | 20 с |
| Поле `minimum` у SOA | 40 с |

## Записи, які потрібно додати (завдання 3)

| Ім'я | Тип | Значення | TTL |
|---|---|---|---|
| `www` | A | `10.53.0.80` | за замовчуванням |
| `api` | CNAME | `www` | за замовчуванням |
| `mail` | A | `10.53.0.80` | за замовчуванням |
| `@` | MX | пріоритет 15, `mail` | за замовчуванням |
| `@` | TXT | `"lab-token=66634402"` | за замовчуванням |
| `test` | A | `10.53.0.81` | **25 с** (вказати явно) |

## URI для розбору (завдання 1)

1. `http://www.se-201-04.lab/orders/archive?q=54&from=asc#results`
2. `http://api.se-201-04.lab:80/archive/?q=%D0%BA%D0%B8%D1%97%D0%B2`
3. `http://user:pass@test.se-201-04.lab/orders#section-2`
4. `https://www.se-201-04.lab:8443/orders/../archive`
5. `mailto:admin@se-201-04.lab?subject=orders`

## Імена для експериментів

- Завдання 4 (TTL): запис `www.se-201-04.lab` переводиться з `10.53.0.80` на `10.53.0.81`.
- Завдання 5 (негативне кешування): ім'я `new-rlwml.se-201-04.lab` спочатку запитується, поки його не існує, і лише потім створюється (A `10.53.0.80`).


| Параметр | Значення |
|---|---|
| `$TTL`, с | |
| TTL запису SOA, с | |
| Поле `minimum` SOA, с | |
| Псевдонім (CNAME на `www`) | |
| Другий хост і його TTL, с | |
| Пріоритет MX | |
| Токен TXT | |
| Ім'я для завдання 5 | |

## Завдання 1. Складові URI

### 1.1. Розбір

<!-- Відсутню складову позначати «—». Порт за замовчуванням записувати з позначкою «(за замовч.)». -->

| № | scheme | userinfo | host | port | path | query | fragment |
|---|---|---|---|---|---|---|---|
| 1 | http | - | www.se-201-04.lab | 80 | /orders/archive | q=54&from=asc | results |
| 2 | http | - | api.se-201-04.lab | 80 | /archive/ | q=%D0%BA%D0%B8%D1%97%D0%B2 | - |
| 3 | http | user:pass | test.se-201-04.lab | 80 | /orders | - | section-2 |
| 4 | https | - | www.se-201-04.lab | (https - 443) в URl - 8443 | /orders/../archive | - | - |
| 5 | mailto | - | - | ? - | admin@se-201-04.lab | subject=orders | - |

| № | Ім'я, що запитується в DNS | Заголовок `Host` | Ціль запиту (request target) |
|---|---|---|---|
| 1 | www.se-201-04.lab | www.se-201-04.lab | /orders/archive?q=54&from=asc#results |
| 2 | api.se-201-04.lab | api.se-201-04.lab | /archive/?q=%D0%BA%D0%B8%D1%97%D0%B2 |
| 3 | test.se-201-04.lab | test.se-201-04.lab | /orders#section-2 |
| 4 | www.se-201-04.lab | www.se-201-04.lab | /orders/../archive |
| 5 | ? - | ? - | - |

Декодоване значення параметра URI 2: **Київ**

Чому URI 5 не має складової host, хоча містить символ `@`: **Символ @ є частиною адреси електронної пошти admin@se-201-04.lab, через це не є складовою host**

### 1.2. Перевірка (після завдання 3)

<!-- Для URI 1–3: рядки запиту (починаються з ">") і тіло відповіді сервера. -->

```text
root@ns-se-201-04:/# curl -v 'http://www.se-201-04.lab/orders/archive?q=54&from=asc#results'
* Host www.se-201-04.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to www.se-201-04.lab (10.53.0.80) port 80
> GET /orders/archive?q=54&from=asc HTTP/1.1
> Host: www.se-201-04.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Thu, 29 Oct 2026 19:35:15 GMT
< Content-Type: text/plain
< Content-Length: 142
< Connection: keep-alive
< 
server: web-a
Host: www.se-201-04.lab
request URI: /orders/archive?q=54&from=asc
path: /orders/archive
query: q=54&from=asc
client: 10.53.1.1
* Connection #0 to host www.se-201-04.lab left intact


root@ns-se-201-04:/# curl -v 'http://api.se-201-04.lab:80/archive/?q=%D0%BA%D0%B8%D1%97%D0%B2'
* Host api.se-201-04.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to api.se-201-04.lab (10.53.0.80) port 80
> GET /archive/?q=%D0%BA%D0%B8%D1%97%D0%B2 HTTP/1.1
> Host: api.se-201-04.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Thu, 29 Oct 2026 19:36:23 GMT
< Content-Type: text/plain
< Content-Length: 156
< Connection: keep-alive
< 
server: web-a
Host: api.se-201-04.lab
request URI: /archive/?q=%D0%BA%D0%B8%D1%97%D0%B2
path: /archive/
query: q=%D0%BA%D0%B8%D1%97%D0%B2
client: 10.53.1.1
* Connection #0 to host api.se-201-04.lab left intact


root@ns-se-201-04:/# curl -v 'http://user:pass@test.se-201-04.lab/orders#section-2'
* Host test.se-201-04.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.81
*   Trying 10.53.0.81:80...
* Connected to test.se-201-04.lab (10.53.0.81) port 80
* Server auth using Basic with user 'user'
> GET /orders HTTP/1.1
> Host: test.se-201-04.lab
> Authorization: Basic dXNlcjpwYXNz
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Thu, 29 Oct 2026 19:37:11 GMT
< Content-Type: text/plain
< Content-Length: 100
< Connection: keep-alive
< 
server: web-b
Host: test.se-201-04.lab
request URI: /orders
path: /orders
query: 
client: 10.53.1.1
* Connection #0 to host test.se-201-04.lab left intact
root@ns-se-201-04:/# 
```

Значення заголовка `Authorization` і результат декодування:

```text
Authorization: Basic dXNlcjpwYXNz
Декодування - через Base64 видало user:pass
```

URI 4: етап, на якому запит не вдається (пошук імені / TCP / TLS), і рядки виводу, що це підтверджують:

```text
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:8443...
* connect to 10.53.0.80 port 8443 from 10.53.1.1 port 52870 failed: Connection refused
* Failed to connect to www.se-201-04.lab port 8443 after 0 ms: Couldn't connect to server
* Closing connection
curl: (7) Failed to connect to www.se-201-04.lab port 8443 after 0 ms: Couldn't connect to server
```

| Запит | `request URI`, отриманий сервером | `path`, отриманий сервером |
|---|---|---|
| звичайний | - | - |
| `--path-as-is` | request URI: /orders/../archive | path: /archive |

Розбіжності між URI та тим, що отримав сервер, і їх пояснення:
```text
В 4 URI відхилено підключення,
в 3 URI Authoriaztion закодовано user:pass через Base64,
В 2 URI Host без 80 порту,
в 1 URI result не передається серверу
```

## Завдання 2. Проходження ієрархії вручну

```text

```

| Сервер | status | flags | ANSWER | AUTHORITY | ADDITIONAL |
|---|---|---|---|---|---|
| 10.53.0.2 | NOERROR | qr | 0 | 1 | 2 |
| 10.53.0.3 | NXDOMAIN | qr aa | 0 | 1 | 1 |
| 10.53.1.1 | REFUSED | qr | 0 | 0 | 1 |

Вивід `dig ns.sNN.lab A +trace`: 

```text
root@ns-se-201-04:/# dig ns.se-201-04.lab A +trace

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> ns.se-201-04.lab A +trace
;; global options: +cmd
.                       86400   IN      NS      a.root-servers.net.
;; Received 59 bytes from 10.53.0.53#53(10.53.0.53) in 320 ms

lab.                    172800  IN      NS      ns1.nic.lab.
;; Received 117 bytes from 10.53.0.2#53(a.root-servers.net) in 80 ms

se-201-04.lab.          3600    IN      NS      ns.se-201-04.lab.
;; Received 103 bytes from 10.53.0.3#53(ns1.nic.lab) in 79 ms

ns.se-201-04.lab.       120     IN      A       10.53.1.1
se-201-04.lab.          120     IN      NS      ns.se-201-04.lab.
;; Received 103 bytes from 10.53.1.1#53(ns.se-201-04.lab) in 0 ms
```

| Запит | TTL запису NS | Прапорець `aa` | Секція, у якій міститься NS |
|---|---|---|---|
| `dig @10.53.0.3 se-201-04.lab NS +norec` | 3600 | - тільки qr | AUTHORITY |
| `dig @10.53.1.1 se-201-04.lab NS +norec` | 120 | \/ | ANSWER |

1. Які відповіді є відсиланнями і за якими ознаками це визначено? **Відсиланням є відповідь DNS-сервера, яка не містить запитаної IP-адреси, а вказує на інший DNS-сервер, до якого потрібно звернутися для отримання подальшої інформації. У результаті команди - повернулося у секції AUTHORITY NS-запис lab. IN NS ns1.nic.lab., а в секції ADDITIONAL — IP-адресу 10.53.0.3 цього сервера.**

2. Для чого у відповіді TLD присутній запис у секції ADDITIONAL і що сталося б, якби його не було? **Запис у секції ADDITIONAL містить IP-адресу DNS-сервера, зазначеного в NS-записі. Він потрібен, щоб клієнт міг звернутися до наступного DNS-сервера, не виконуючи додатковий запит для визначення його IP-адреси.**

3. Яка з двох копій запису NS є авторитативною і чому? **Авторитативною є копія NS-запису, що міститься в зоні на сервері, який відповідає за цю зону. Копія NS-запису в батьківській зоні використовується для делегування повноважень дочірній зоні.**

## Завдання 3. Заповнення власної зони

Вивід `zone-reload`:

```text
zone-reload
zone se-201-04.lab/IN: loaded serial 2026102201
OK
zone reload up-to-date
```

<!-- Шість запитів до 10.53.1.1 з +norec, по одному на кожен рядок таблиці картки. -->

| Запис | status | Прапорець `aa` | Значення в ANSWER | TTL | Відповідає картці (так/ні) |
|---|---|---|---|---|---|
| `www` A | NOERROR | \/ | 10.53.0.80 | 120 | так |
| псевдонім CNAME | NOERROR | \/ | www.se-201-04.lab. | 120 | так |
| `mail` A | NOERROR | \/ | 10.53.0.80 | 120 | так |
| `@` MX | NOERROR | \/ | 15 mail.se-201-04.lab. | 120 | так |
| `@` TXT | NOERROR | \/ | "lab-token=66634402" | 120 | так |
| другий хост A | NOERROR | \/ | 10.53.0.81 | 25 | так |

Порівняння відповідей резолвера та власного сервера:

```text
Повертається відповідь з доп значеннями у flags, а саме rd і ra
що використовуються при рекурсії і її успішність 
```

| Запит | flags | ANSWER (кількість, типи) | AUTHORITY (кількість, типи) | ADDITIONAL (кількість, типи) | TTL |
|---|---|---|---|---|---|
| псевдонім через resolver-a | qr rd ra | 2 CNAME, A | 0 | 1 OPT? | 120 |
| псевдонім через 10.53.1.1 | qr aa | 2 CNAME, A | 1 NS | 2 A OPT? | 120 |
| MX через resolver-a | qr rd ra | 1 MX | 0 | 1 OPT? | 120 |
| MX через 10.53.1.1 | qr aa | 1 MX | 1 NS | 3 A A OPT? | 120 |

Пояснення відмінностей:

Повідомлення `zone-check` після тимчасового додавання `@ IN CNAME www`:

```text
root@ns-se-201-04:/# zone-check
dns_master_load: /lab/zone/se-201-04.lab.zone:21: se-201-04.lab: CNAME and other data
zone se-201-04.lab/IN: loading from master file /lab/zone/se-201-04.lab.zone failed: CNAME and other data
zone se-201-04.lab/IN: not loaded due to errors.
```

1. Який TTL має кожен запис зони і звідки це значення взято? ** Самі прописуємо через zone edit у багатьох випадках 120 секунд і вказуємо для яких записів яке значення **

2. Скільки записів містить секція ANSWER у відповіді на запит псевдоніма і чому? ** Секція ANSWER має 2 записи CNAME і А, CNAME говорить нам про те що він є www.se-201-04.lab, А - IP адресу** 

3. Чому запис CNAME неможливо розмістити для імені `@`? ** CNAME неможливо розмістити для імені `@` тому що якщо вже є запис с CNAME то не може бути їх 2 **

## Завдання 4. TTL і видимість змін

Два запити з інтервалом 5 с:

```text
root@ns-se-201-04:/# dig www.se-201-04.lab +noall +answer
www.se-201-04.lab.      120     IN      A       10.53.0.80
root@ns-se-201-04:/# dig www.se-201-04.lab +noall +answer
www.se-201-04.lab.      110     IN      A       10.53.0.80
```

**Прогноз** (записати до вимірювання). Формула часу появи нової адреси через залишок R, найбільша можлива затримка: ** t + R = t new **

Перевірка, що сервер до `zone-reload` повертає стару адресу:

```text
root@ns-se-201-04:/# dig @10.53.1.1 www.se-201-04.lab +norec +short
10.53.0.80
```

Залишок R і час (крок 4), час `zone-reload` (крок 5), перевірка на сервері (крок 6):

```text
root@ns-se-201-04:/# date +%T; dig www.se-201-04.lab +noall +answer
20:19:47
www.se-201-04.lab.      120     IN      A       10.53.0.80

root@ns-se-201-04:/# zone-reload; date +%T 
zone se-201-04.lab/IN: loaded serial 2026102202
OK
zone reload up-to-date
20:20:22

root@ns-se-201-04:/# dig @10.53.1.1 www.se-201-04.lab +norec +short 
10.53.0.81
```

Очікуваний час появи нової адреси (підставити R у формулу): ** 20:21:47 **

Вивід циклу спостереження (перші два рядки, два останні рядки зі старою адресою, рядок з новою адресою):

```text
P.S. Трохи протупив і запустив пізніше +- на нестачу секунд щоб було 2 хвилини на оновлення
root@ns-se-201-04:/# while true; do echo "$(date +%T) $(dig +short www.se-201-04.lab)"; sleep 2; done
20:35:35 10.53.0.80
20:35:37 10.53.0.80

20:36:39 10.53.0.80
20:36:41 10.53.0.81
```

Рядок `server:` з відповіді `curl`: ** server: web-b **

| Час зміни | Очікуваний час | Спостережений час | Виміряна затримка, с | Прогноз, с |
|---|---|---|---|---|
| 20:35:11 | 20:37:11 | 20:36:41 | 90 | 120 |

1. Чим пояснюється розбіжність між прогнозом і виміряним значенням? ** Розбіжність пояснюється тим, що на момент зміни запису залишалося менше ніж 120 секунд **

2. Якою була б найбільша можлива затримка для цього запису і за яких умов? ** 120 секунд **

3. Як слід діяти, щоб перенести сервіс на нову адресу, якщо TTL запису становить 86 400 с?
```text
Спочатку зменшити TTL до тих самих 120 секунд, і дочекатися, поки попередній TTL пройде. Потім змінити IP-адресу та перезавантажити зону через zone-reload І потім якщо потрібно повернути до 86 400 секунд значення TTL.
```
## Завдання 5. Негативне кешування

**Прогноз** (записати до вимірювання). Формула за RFC 2308 і розраховане значення:
```text
Negative Cache TTL = min(SOA_TTL, MINIMUM) формула за RFC 2308

В моєму випадку це буде 20 секунд

| TTL самого запису SOA | 20 с |
| Поле `minimum` у SOA | 40 с |
```

Перевірка, що сервер до `zone-reload` повертає `NXDOMAIN`:

```text
root@ns-se-201-04:/dig @10.53.1.1 new-rlwml.se-201-04.lab +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 new-rlwml.se-201-04.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 1091
;; flags: qr aa; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: a3835c2ed76d464f010000006ae3b3b9e2c1c79e543d4968 (good)
;; QUESTION SECTION:
;new-rlwml.se-201-04.lab.       IN      A

;; AUTHORITY SECTION:
se-201-04.lab.          20      IN      SOA     ns.se-201-04.lab. admin.se-201-04.lab. 2026102204 3600 600 86400 40

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Thu Oct 29 20:59:37 UTC 2026
;; MSG SIZE  rcvd: 125
```

Запит через resolver-a (крок 3), час `zone-reload` (крок 4), перевірка на сервері (крок 5):

```text
root@ns-se-201-04:/# date +%T; dig new-rlwml.se-201-04.lab
21:00:49

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> new-rlwml.se-201-04.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 6496
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;new-rlwml.se-201-04.lab.       IN      A

;; AUTHORITY SECTION:
se-201-04.lab.          20      IN      SOA     ns.se-201-04.lab. admin.se-201-04.lab. 2026102204 3600 600 86400 40

;; Query time: 1 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Thu Oct 29 21:00:49 UTC 2026
;; MSG SIZE  rcvd: 97

root@ns-se-201-04:/#  zone-reload; date +%T
zone se-201-04.lab/IN: loaded serial 2026102205
OK
zone reload queued
21:00:50

root@ns-se-201-04:/# dig @10.53.1.1 new-rlwml.se-201-04.lab +norec +short
10.53.0.80
```

Вивід циклу спостереження (перші два рядки, останній рядок з `NXDOMAIN`, перший рядок з `NOERROR`):

```text
21:00:54 status: NXDOMAIN
21:00:56 status: NXDOMAIN

21:01:08 status: NXDOMAIN
21:01:10 status: NOERROR
```

| Негативну відповідь закешовано | Запис створено | Ім'я резолвиться через resolver-a | Виміряно, с | Прогноз, с |
|---|---|---|---|---|
| 21:00:54 | 21:00:50 | 21:01:10 | 16 | 20 |

<!-- Якщо завдання повторювалося з іншим ім'ям (new2-…), зазначити це й причину. -->

1. Чому тривалість негативного кешування визначається записом SOA, а не записом, який запитували? ** Запис не існує, тому не має власного TTL і тривалість визначається SOA **

2. Чи збігається TTL запису SOA в секції AUTHORITY з TTL запису SOA у файлі зони? Поясніть. ** І так і ні в різниці від заданих значень часу minimum і TTL **

3. Скрипт розгортання перевіряє ім'я сервісу до створення запису, а потім одразу після. Що він побачить і як це виправити? ** Спочатку скрипт видає NXDOMAIN, після створення запису може далі отримувати NXDOMAIN через негативне кешування виправляється це через повторення запитів з деяким інтервалом і врешті дочекатися завершення кешування **

## Завдання 6. Затримка пошуку імені

**Прогноз** (записати до вимірювання). До яких серверів звернеться resolver-a:

| Ім'я | Перший запит | Повторний запит |
|---|---|---|
| A: нове ім'я у власній зоні | сервер se-201-04.lab | кеш resolver |
| B: нове ім'я в зоні `lab.` | зона snn.lab | кеш resolver |
| C: ім'я в неіснуючому TLD | кореневий сервер | кеш resolver |
| D: `www.sNN.lab` після перезапуску резолвера | той сервер який буде питатися | кеш resolver |

Вивід вимірювань:

```text
root@ns-se-201-04:/# A=y$RANDOM.sNN.lab; dig $A | grep 'Query time'; dig $A | grep 'Query time'
B=x$RANDOM.lab; dig $B | grep 'Query time'; dig $B | grep 'Query time'
C=x$RANDOM.zz$RANDOM; dig $C | grep 'Query time'; dig $C | grep 'Query time'
;; Query time: 39 msec
;; Query time: 0 msec
;; Query time: 40 msec
;; Query time: 0 msec
;; Query time: 81 msec
;; Query time: 0 msec

root@ns-se-201-04:/# A=y$RANDOM.sNN.lab; dig $A | grep 'Query time'; dig $A | grep 'Query time'
B=x$RANDOM.lab; dig $B | grep 'Query time'; dig $B | grep 'Query time'
C=x$RANDOM.zz$RANDOM; dig $C | grep 'Query time'; dig $C | grep 'Query time'
;; Query time: 40 msec
;; Query time: 1 msec
;; Query time: 41 msec
;; Query time: 0 msec
;; Query time: 80 msec
;; Query time: 0 msec
root@ns-se-201-04:/# 
```

| Ім'я | Перший запит, мс | Повторний, мс | Сервери, до яких звернувся резолвер (за журналами) |
|---|---|---|---|
| A | 39 | 0 | сервер se-201-04.lab |
| B | 40 | 0 | зона snn.lab |
| C | 81 | 0 | кореневий сервер |
| D | 403 | 0 | кореневий сервер і сервер snn.lab  |

Значення `time_namelookup` з `curl` після перезапуску резолвера:

Фрагменти журналів запитів root, TLD і власного сервера для рядка D:

```text
@DanDytiuk ➜ /workspaces/opism-lab-DNSMy (master) $ docker compose -f stand/compose.yaml exec root tail /var/l
og/named/queries.log
30-Oct-2026 13:14:08.599 client @0x76d104006078 10.53.0.53#60616 (.): query: . IN NS -E(0)DC (10.53.0.2)
30-Oct-2026 13:14:08.680 client @0x76d0fc06ac88 10.53.0.53#13061 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
30-Oct-2026 13:14:08.760 client @0x76d104006078 10.53.0.53#53396 (ns.se-201-04.lab): query: ns.se-201-04.lab IN A -E(0)DC (10.53.0.2)
30-Oct-2026 13:14:09.264 client @0x76d104006078 10.53.0.54#28073 (.): query: . IN NS -E(0)DC (10.53.0.2)
30-Oct-2026 13:14:09.345 client @0x76d0fc06ac88 10.53.0.54#39768 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
30-Oct-2026 13:14:09.425 client @0x76d0fc06ac88 10.53.0.54#57327 (ns.se-201-04.lab): query: ns.se-201-04.lab IN A -E(0)DC (10.53.0.2)
@DanDytiuk ➜ /workspaces/opism-lab-DNSMy (master) $ docker compose -f stand/compose.yaml exec tld tail /var/log/named/queries.log
30-Oct-2026 13:14:08.881 client @0x7a6910005268 10.53.0.53#28464 (ns.se-201-04.lab): query: ns.se-201-04.lab IN A -E(0)DC (10.53.0.3)
30-Oct-2026 13:14:09.546 client @0x7a6910005268 10.53.0.54#65070 (ns.se-201-04.lab): query: ns.se-201-04.lab IN A -E(0)DC (10.53.0.3)
```

1. Яку затримку вносить кожен із серверів root, TLD і власний сервер? Покажіть, з яких рядків таблиці це випливає. ** 39 , 40, 81 нове ім'я займає менеше часу на обробку **

2. Чому в рядку A перший запит не проходить через root і TLD, хоча ім'я нове? ** Нове лише доменне ім'я, а інформація про делегування se-201-04.lab уже збережена в кеші resolver **

3. Вебсторінка завантажує ресурси з п'яти різних доменів. Як це впливає на час першого відкриття сторінки? ** На це впливає фактор того що DNS пошук для кожної сторінки повинен бути **

## Завдання 7. Інцидент: зупинка власного сервера

**Прогноз** (записати до вимірювання). Що поверне кожен резолвер після зупинки сервера та спливання TTL і скільки часу займе відповідь:

Кешування запису в обох резолверах (крок 2) і час зупинки сервера (крок 3):

```text

```

Запити із заміром часу (крок 5):

```text

```

| Запит | status або тайм-аут | ANSWER, TTL | `real` |
|---|---|---|---|
| resolver-a, перший | | | |
| resolver-a, другий | | | |
| resolver-b | | | |

Час `named-start` і вивід циклу спостереження до першої успішної відповіді (крок 6):

```text

```

Перевірка після завдання (`dig @10.53.1.1 sNN.lab SOA +norec +short`):

```text

```

### Постмортем

1. Що сталося:
2. Вплив на клієнтів resolver-a і resolver-b:
3. Хронологія з виміряним часом:
4. Чому кеш допоміг або не допоміг:
5. Що слід змінити:

1. Який TTL мав запис у відповіді resolver-b і навіщо такий TTL?

2. Порівняйте результат і тривалість двох запитів до resolver-a. Якщо вони різняться, сформулюйте гіпотезу щодо причини; якщо ні, поясніть, що робить резолвер протягом цього часу.

3. Чому після `named-start` ім'я через resolver-a зарезолвилося не одразу?

4. У якій ситуації serve-stale шкодить, а не допомагає? Пов'яжіть відповідь із завданням 4.

## Фінальний файл зони

<!-- Повний вміст stand/students/sNN/zone/sNN.lab.zone після завершення роботи. -->

```text

```

## Висновки

**Що виявилося несподіваним.** Результат, який не збігся з очікуванням, і рядок виводу, де це видно:

**Яке питання залишилося без відповіді.** Що саме не вдалося пояснити; до якого рядка виводу це стосується; що вже перевірено (RFC, документація, додатковий експеримент на стенді) і що з'ясувати не вдалося:

## Декларація використання ШІ

Використовувалися інструменти ШІ: <!-- так / ні -->

<!-- Якщо так, заповнити таблицю: один рядок на кожне звернення, що вплинуло на звіт. Промпт наводити дослівно. -->

| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 | | | |
