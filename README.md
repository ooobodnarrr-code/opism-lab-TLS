<!--
Бланк звіту до лабораторної роботи «TLS та сертифікати».
Скопіювати цей файл у власний публічний репозиторій звіту як README.md і заповнити.
Вивід команд вставляти текстом у блоки коду, а не знімками екрана.
Порожні поля не допускаються: якщо щось не вдалося, записати, що саме і чому.
-->

# Лабораторна робота 2. TLS та сертифікати

| Поле | Значення |
|---|---|
| ПІБ |Боднар Олександра |
| Група | 2.02|
| Ідентифікатор |cs-202-02 |
| Дата виконання |12.11.2026 |
| Середовище | GitHub Codespacesе |
| Версія OpenSSL |OpenSSL 3.0.13 30 Jan 2024|

## Параметри варіанта

<!-- Переписати з картки stand/cards/<id>.md. -->

| Параметр | Значення |
|---|---|
| Віддалений сервер | origin.cs-202-02.lab|
| Імена для сертифіката |www.cs-202-02.lab, status.cs-202-02.lab |
| Ваш сервер (IP) |10.53.1.1 |

<!-- Якщо стенд перезапускався під час роботи, вказати, після якого завдання. -->

## Завдання 1. HTTP і HTTPS у дампі трафіку

Вивід кроку 1.1:

```
[1] 89
server: web
Host: origin.cs-202-02.lab
SNI: 
protocol: HTTP/1.1 
client: 10.53.1.1
[1]+  Exit 124                timeout 8 tcpdump -ni eth0 -A -s0 -l 'host 10.53.0.80 and tcp port 80' > t1-http.txt 2>&1
18:10:36.347218 IP 10.53.1.1.33882 > 10.53.0.80.80: Flags [P.], seq 1:84, ack 1, win 502, options [nop,nop,TS val 694166405 ecr 4210517694], length 83: HTTP: GET / HTTP/1.1
)`#...f.GET / HTTP/1.1
Host: origin.cs-202-02.lab
server: web

```

Вивід кроку 1.2:

```
[1] 95
server: web
Host: origin.cs-202-02.lab
SNI: origin.cs-202-02.lab
protocol: HTTP/2.0 TLSv1.3
client: 10.53.1.1
[1]+  Exit 124                timeout 8 tcpdump -ni eth0 -A -s0 -l 'host 10.53.0.80 and tcp port 443' > t1-https.txt 2>&1
Кількість відкритих рядків GET і server:
0
Кількість входжень імені сервера:
1

```

| Що шукали в дампі | HTTP | HTTPS |
|---|---|---|
| рядок запиту `GET /` | GET / HTTP/1.1|0 входжень|
| заголовок `Host` | Host: origin.cs-202-02.lab|- |
| тіло відповіді (`server: web`) |server: web |0 входжень|
| ім'я віддаленого сервера (кількість входжень) | origin.cs-202-02.lab — 1 раз|origin.cs-202-02.lab — 1 раз |

1. У якому повідомленні і в якому полі передається ім'я сервера, яке видно в дампі HTTPS?

Ім'я сервера передається в повідомленні ClientHello під час встановлення TLS-з'єднання. Для цього використовується розширення server_name (SNI). У нашому випадку в ньому вказано origin.cs-202-02.lab. Це підтверджується виводом кроку 1.2, де поле SNI містить це ім'я. У дампі ім'я сервера зустрічається один раз.

2. Чому рядок `SNI:` у відповіді сервера порожній для HTTP і заповнений для HTTPS?

Під час HTTP-з'єднання TLS не використовується, тому клієнт не передає ім'я сервера через SNI. Саме тому вивід показує порожнє поле SNI:. У випадку HTTPS спочатку встановлюється захищене TLS-з'єднання, під час якого клієнт передає ім'я сервера. У нашому результаті це origin.cs-202-02.lab. Це також підтверджує, що для HTTPS використовується TLS 1.3

3. Рядок `protocol:` показує HTTP/1.1 для запиту по HTTP і HTTP/2.0 для запиту по HTTPS, хоча обидва запити виконано однією командою `curl` без додаткових ключів. Що визначило версію HTTP у другому випадку?

Версію HTTP визначило узгодження протоколу за допомогою ALPN під час TLS-рукостискання. Цей механізм дозволяє клієнту та серверу домовитися, яку версію HTTP використовувати. У нашому випадку для HTTPS вони узгодили HTTP/2.0, що видно у виводі protocol: HTTP/2.0 TLSv1.3. Для звичайного HTTP використовувався HTTP/1.1. Отже, хоча команда curl була однаковою, для HTTPS було узгоджено іншу версію протоколу.

## Завдання 2. Рукостискання по записах

Вивід кроку 2.1 (TLS 1.3):

```
Sent Record
  Content Type = Handshake (22)
    ClientHello, Length=331
Received Record
  Content Type = Handshake (22)
    ServerHello, Length=118
Received Record
  Content Type = ChangeCipherSpec (20)
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    EncryptedExtensions, Length=15
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    Certificate, Length=990
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    CertificateVerify, Length=76
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    Finished, Length=48
Sent Record
  Content Type = ChangeCipherSpec (20)
Sent Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    Finished, Length=48
Sent Record
  Content Type = ApplicationData (23)
  Inner Content Type = Alert (21)
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
ALPN protocol: h2
Verify return code: 0 (ok)

```

Вивід кроку 2.2 (TLS 1.2):

```
Sent Record
  Content Type = Handshake (22)
    ClientHello, Length=226
Received Record
  Content Type = Handshake (22)
    ServerHello, Length=74
Received Record
  Content Type = Handshake (22)
    Certificate, Length=985
Received Record
  Content Type = Handshake (22)
    ServerKeyExchange, Length=111
Received Record
  Content Type = Handshake (22)
    ServerHelloDone, Length=0
Sent Record
  Content Type = Handshake (22)
    ClientKeyExchange, Length=33
Sent Record
  Content Type = ChangeCipherSpec (20)
Sent Record
  Content Type = Handshake (22)
    Finished, Length=12
Received Record
  Content Type = Handshake (22)
    NewSessionTicket, Length=230
Received Record
  Content Type = ChangeCipherSpec (20)
Received Record
  Content Type = Handshake (22)
    Finished, Length=12
Received Record
  Content Type = ApplicationData (23)
Sent Record
  Content Type = Alert (21)
New, TLSv1.2, Cipher is ECDHE-ECDSA-AES256-GCM-SHA384
ALPN protocol: h2
    Verify return code: 0 (ok)

```

<!-- Один рядок на кожен запис з виводу 2.1, у тому самому порядку. Напрям: К→С або С→К. -->

| № | Напрям | `Content Type` | `Inner Content Type` | Повідомлення | `Length` |
|---|---|---|---|---|---|
| 1 |К→С|Handshake (22) |- |ClientHello|331 |
| 2 |С→К |Handshake (22) |- | ServerHello|118 |
| 3 |С→К|ChangeCipherSpec (20) |-|ChangeCipherSpec |-|
| 4 |С→К |ApplicationData (23)|Handshake (22) |EncryptedExtensions|15|
| 5 |С→К|ApplicationData (23) |Handshake (22) |	Certificate| 990|
| 6 |С→К|ApplicationData (23) |Handshake (22) |	CertificateVerify |76|
| 7 |С→К|ApplicationData (23) |Handshake (22)|	Finished| 48|
| 8 | К→С |ChangeCipherSpec (20)|- | ChangeCipherSpec| -|
| 9 | К→С |ApplicationData (23) |Handshake (22) |	Finished | 48|
| 10 |К→С |ApplicationData (23)|Alert (21) | 	Alert| -|

Вивід кроків 2.3 і 2.4:

```
root@cs-202-02:/lab/work# grep -a -A3 'extension_type=server_name' t2-13.txt | head -4
        extension_type=server_name(0), length=25
          0000 - 00 17 00 00 14 6f 72 69-67 69 6e 2e 63 73 2d   .....origin.cs-
          000f - 32 30 32 2d 30 32 2e 6c-61 62                  202-02.lab
        extension_type=ec_point_formats(11), length=4

root@cs-202-02:/lab/work# for v in 1.3 1.2; do
  timeout 6 tcpdump -ni eth0 -A -s0 -l 'host 10.53.0.80 and tcp port 443' > t2-dump-$v.txt 2>&1 &
  sleep 1
  curl -s -o /dev/null --tlsv1.2 --tls-max $v --cacert $STAND_CA https://origin.cs-202-02.lab/
  wait
  echo "TLS $v: $(grep -a -c 'OPISM Stand Intermediate' t2-dump-$v.txt)"
done
[1] 114
[1]+  Exit 124                timeout 6 tcpdump -ni eth0 -A -s0 -l 'host 10.53.0.80 and tcp port 443' > t2-dump-$v.txt 2>&1
TLS 1.3: 0
[1] 120
[1]+  Exit 124                timeout 6 tcpdump -ni eth0 -A -s0 -l 'host 10.53.0.80 and tcp port 443' > t2-dump-$v.txt 2>&1
TLS 1.2: 2

```

| Показник | TLS 1.3 | TLS 1.2 |
|---|---|---|
| Довжина ClientHello, байт | 331|226 |
| Номер першого запису, тип якого спостерігач не бачить | 4|8 |
| Кількість RTT від ClientHello до першого HTTP-запиту |1 RTT |2 RTT |
| Чи видно сертифікат у дампі (результат 2.4) | ні|так|
| Обраний протокол ALPN |h2 |h2 |

1. За якими рядками виводу 2.1 визначено, з якого запису починається шифрування? Що бачить спостерігач замість типу кожного наступного запису?

```
Початок шифрування визначено за четвертим записом, у якому вказано Content Type = ApplicationData (23), а всередині нього — Inner Content Type = Handshake (22). Це означає, що повідомлення рукостискання вже передаються в зашифрованому вигляді. З цього моменту спостерігач не може побачити справжній тип внутрішнього повідомлення без ключів шифрування. Замість нього він бачить зовнішній тип ApplicationData (23). Наприклад, у нашому виводі повідомлення Certificate, CertificateVerify і Finished мають саме такий зовнішній тип.
```
2. Чому ClientHello у TLS 1.3 довший, ніж у TLS 1.2? Назвіть розширення, яке за рахунок цього скорочує рукостискання на один RTT.

```
У виводі ClientHello для TLS 1.3 має довжину 331 байт, а для TLS 1.2 — 226 байт. Він довший, тому що TLS 1.3 використовує додаткові розширення, які допомагають швидше узгодити параметри захищеного з'єднання. Одним із важливих є supported_versions, яке дозволяє клієнту вказати підтримувані версії TLS. Також використовується key_share, завдяки якому клієнт одразу передає свої параметри обміну ключами. Саме key_share дає змогу скоротити рукостискання на один RTT, оскільки не потрібно чекати окремого повідомлення для обміну параметрами ключів.
```
3. Довжина розширення `server_name` у виводі 2.3 становить 17 байт для імені з 12 символів. З яких частин складається це значення і яким воно буде для вашого імені?

```
Довжина розширення складається з довжини самого доменного імені та службових байтів, які описують його тип і довжину. Для імені з 12 символів отримуємо 17 байт: 12 байт на саме ім'я та 5 службових байтів. У нашому випадку ім'я сервера — origin.cs-202-02.lab, яке має 20 символів. Тому довжина розширення server_name для нього становить 25 байт. Це збігається з результатом кроку 2.3, де вказано length=25.
```

4. Навіщо в TLS 1.3 передається запис ChangeCipherSpec, якщо в рукостисканні він нічого не змінює?

```
У TLS 1.3 запис ChangeCipherSpec передається переважно для сумісності зі старими реалізаціями TLS, які очікують побачити таке повідомлення під час рукостискання. Він не змінює параметри шифрування і не впливає на встановлення захищеного з'єднання. У нашому виводі TLS 1.3 цей запис видно двічі: один раз від сервера і один раз від клієнта. При цьому основні повідомлення рукостискання передаються через ApplicationData із зашифрованим внутрішнім вмістом. Тобто ChangeCipherSpec тут має допоміжне призначення, а не виконує реальне перемикання шифрування.
```
## Завдання 3. Сертифікат для власного сервера

Вивід кроків 3.1 і 3.2:

```
root@cs-202-02:/lab/work# acme-root
збережено /lab/tls/acme-root.crt

root@cs-202-02:/lab/work# openssl x509 -in /lab/tls/acme-root.crt -noout -subject -dates
subject=CN = Pebble Root CA 099713
notBefore=Nov 12 17:30:08 2026 GMT
notAfter=Nov 12 17:30:08 2056 GMT

root@cs-202-02:/lab/work# curl -sS --cacert /lab/tls/acme-root.crt https://www.cs-202-02.lab/
curl: (60) SSL certificate problem: self-signed certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.

```

Вивід кроку 3.3 (останні рядки `lego` і рядки журналу nginx):

```
2026/11/12 18:50:13 [INFO] [www.cs-202-02.lab, status.cs-202-02.lab] acme: Obtaining SAN certificate
2026/11/12 18:50:13 [INFO] [www.cs-202-02.lab] AuthURL: https://ca.lab:14000/authZ/GHjj-N7ZSTwl-ULecyv4QRQxxnbX5conRRm92kO8Dw8
2026/11/12 18:50:13 [INFO] [status.cs-202-02.lab] AuthURL: https://ca.lab:14000/authZ/W-fvgyxCbLInWR3sKh8n_PUHQS_krAGYOdxM1XbXjIQ
2026/11/12 18:50:13 [INFO] [www.cs-202-02.lab] acme: Could not find solver for: tls-alpn-01
2026/11/12 18:50:13 [INFO] [www.cs-202-02.lab] acme: use http-01 solver
2026/11/12 18:50:13 [INFO] [status.cs-202-02.lab] acme: Could not find solver for: tls-alpn-01
2026/11/12 18:50:13 [INFO] [status.cs-202-02.lab] acme: use http-01 solver
2026/11/12 18:50:13 [INFO] [www.cs-202-02.lab] acme: Trying to solve HTTP-01
2026/11/12 18:50:18 [INFO] [www.cs-202-02.lab] The server validated our request
2026/11/12 18:50:18 [INFO] [status.cs-202-02.lab] acme: Trying to solve HTTP-01
2026/11/12 18:50:24 [INFO] [status.cs-202-02.lab] The server validated our request
2026/11/12 18:50:24 [INFO] [www.cs-202-02.lab, status.cs-202-02.lab] acme: Validations succeeded; requesting certificates
2026/11/12 18:50:24 [INFO] Wait for certificate [timeout: 30s, interval: 500ms]
2026/11/12 18:50:24 [INFO] [www.cs-202-02.lab] Server responded with a certificate.

10.53.0.10 - - [12/Nov/2026:18:50:13 +0000] "GET /.well-known/acme-challenge/OtNa8xqXPTzDTEYMlLe26iPJM3Ql_R7JKbPQSSbJQOQ HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [12/Nov/2026:18:50:13 +0000] "GET /.well-known/acme-challenge/OtNa8xqXPTzDTEYMlLe26iPJM3Ql_R7JKbPQSSbJQOQ HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [12/Nov/2026:18:50:13 +0000] "GET /.well-known/acme-challenge/OtNa8xqXPTzDTEYMlLe26iPJM3Ql_R7JKbPQSSbJQOQ HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [12/Nov/2026:18:50:18 +0000] "GET /.well-known/acme-challenge/DoQ81I54cHFy0yDU4Rf1LTjgebD2KrB1Adnp-DImib4 HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [12/Nov/2026:18:50:18 +0000] "GET /.well-known/acme-challenge/DoQ81I54cHFy0yDU4Rf1LTjgebD2KrB1Adnp-DImib4 HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [12/Nov/2026:18:50:18 +0000] "GET /.well-known/acme-challenge/DoQ81I54cHFy0yDU4Rf1LTjgebD2KrB1Adnp-DImib4 HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"

```

Вивід кроку 3.4:

```
root@cs-202-02:/lab/work/lego/certificates# openssl x509 -in www.cs-202-02.lab.crt -noout -subject -issuer -dates -serial -ext subjectAltName
subject=CN = www.cs-202-02.lab
issuer=CN = Pebble Intermediate CA 41d7f1
notBefore=Nov 12 18:50:24 2026 GMT
notAfter=Nov 12 18:50:23 2031 GMT
serial=692C82380170E5BC
X509v3 Subject Alternative Name:
    DNS:www.cs-202-02.lab, DNS:status.cs-202-02.lab

root@cs-202-02:/lab/work/lego/certificates# openssl x509 -in www.cs-202-02.lab.issuer.crt -noout -subject -issuer
subject=CN = Pebble Intermediate CA 41d7f1
issuer=CN = Pebble Root CA 099713

```

Вивід кроку 3.5:

```
nginx: the configuration file /lab/nginx.conf syntax is ok
nginx: configuration file /lab/nginx.conf test is successful
2026/11/12 18:52:27 [notice] 143#143: signal process started

curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.

Certificate chain
 0 s:CN = www.cs-202-02.lab
   i:CN = Pebble Intermediate CA 41d7f1
   a:PKEY: id-ecPublicKey, 256 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 12 18:50:24 2026 GMT; NotAfter: Nov 12 18:50:23 2031 GMT
---
Verify return code: 21 (unable to verify the first certificate)

```

Вивід кроку 3.6:

```
nginx: the configuration file /lab/nginx.conf syntax is ok
nginx: configuration file /lab/nginx.conf test is successful
2026/11/12 18:52:47 [notice] 151#151: signal process started

server: cs-202-02 (https)
Host: www.cs-202-02.lab
SNI: www.cs-202-02.lab
protocol: TLSv1.3

Certificate chain
 0 s:CN = www.cs-202-02.lab
   i:CN = Pebble Intermediate CA 41d7f1
   a:PKEY: id-ecPublicKey, 256 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 12 18:50:24 2026 GMT; NotAfter: Nov 12 18:50:23 2031 GMT
 1 s:CN = Pebble Intermediate CA 41d7f1
   i:CN = Pebble Root CA 099713
   a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 12 17:30:08 2026 GMT; NotAfter: Nov 12 17:30:08 2056 GMT
---
Verify return code: 0 (ok)

server: cs-202-02 (https)
Host: status.cs-202-02.lab
SNI: status.cs-202-02.lab
protocol: TLSv1.3

```

| Параметр | Значення |
|---|---|
| Видавець сертифіката сервера |Pebble Intermediate CA 41d7f1|
| Видавець сертифіката видавця |Pebble Root CA 099713|
| Імена в SAN |	www.cs-202-02.lab, status.cs-202-02.lab|
| Строк дії (від – до) |12.11.2026 – 12.11.2031 |
| Тип ключа сервера і алгоритм підпису |EC, 256 біт; RSA-SHA256 |
| Кількість сертифікатів, які надсилає сервер, у кроці 3.5 і в кроці 3.6 |3.5 — 1 сертифікат; 3.6 — 2 сертифікати |

1. Хто і як перевірив, що ви контролюєте обидва імена? Вкажіть рядки виводу, які це підтверджують.
```
Перевірку виконав центр сертифікації Pebble за допомогою протоколу ACME. Для цього було використано метод HTTP-01, який передбачає розміщення спеціального перевірочного файлу на сервері. Центр сертифікації звернувся до обох доменів і перевірив доступність цих файлів. У виводі це підтверджують рядки The server validated our request для www.cs-202-02.lab та status.cs-202-02.lab. Також у журналі nginx видно запити від адреси 10.53.0.10 до шляху /.well-known/acme-challenge/, які завершилися кодом 200. Це означає, що перевірка обох імен пройшла успішно.
```
2. Чому в кроці 3.5 перевірка не пройшла, хоча корінь центру сертифікації вказано в `--cacert`, а сертифікат виданий правильно?
```
Перевірка не пройшла через те, що сервер передавав тільки власний сертифікат, без проміжного сертифіката центру сертифікації. У виводі кроку 3.5 видно лише один сертифікат — www.cs-202-02.lab, виданий Pebble Intermediate CA 41d7f1. Хоча кореневий сертифікат було вказано через --cacert, клієнту все одно потрібно отримати проміжний сертифікат, щоб побудувати повний ланцюжок довіри. Це підтверджує помилка Verify return code: 21 (unable to verify the first certificate). Після додавання проміжного сертифіката в кроці 3.6 перевірка пройшла успішно.
```
3. У виводі ланцюжка для сертифіката сервера вказано `id-ecPublicKey` і `RSA-SHA256`. Що означає кожне з цих значень і чому вони не суперечать одне одному?
```
id-ecPublicKey означає, що відкритий ключ сервера створений на основі еліптичних кривих (EC). У нашому випадку його довжина становить 256 біт. RSA-SHA256 означає, що для підпису сертифіката використано алгоритм RSA разом із хеш-функцією SHA-256. Ці значення не суперечать одне одному, оскільки описують різні речі. Перше вказує на тип відкритого ключа сервера, а друге — на алгоритм, яким центр сертифікації підписав сертифікат. Тому сервер може використовувати EC-ключ, а сертифікат при цьому може бути підписаний за допомогою RSA-SHA256.
```
4. Що станеться з перевіркою вашого сервера після перезапуску стенда і чому? Що в реальній інфраструктурі відповідає цій ситуації?
```
Після перезапуску стенда перевірка сервера може знову завершитися помилкою, оскільки створені під час роботи сертифікати та налаштування можуть не зберегтися. Якщо сертифікат буде втрачено або сервер повернеться до початкового самопідписаного сертифіката, клієнт не зможе перевірити його довіру. У реальній інфраструктурі подібна ситуація може виникнути під час перевстановлення сервера, втрати файлів сертифіката або неправильного відновлення резервної копії. Щоб цього уникнути, потрібно зберігати сертифікати, приватні ключі та налаштування, а також налаштовувати автоматичне поновлення сертифікатів.
```
## Завдання 4. Діагностика несправностей

Вивід кроку 4.1:

```
=== f1.cs-202-02.lab
curl: (60) SSL: no alternative certificate subject name matches target host name 'f1.cs-202-02.lab'
Certificate chain
 0 s:CN = legacy.cs-202-02.lab
   i:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   v:NotBefore: Nov 12 17:30:07 2026 GMT; NotAfter: Feb 10 17:30:07 2027 GMT
 1 s:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   i:O = OPISM Lab, CN = OPISM Stand Root CA
   v:NotBefore: Nov 12 17:30:07 2026 GMT; NotAfter: Nov 11 17:30:07 2031 GMT
---
Verify return code: 0 (ok)
=== f2.cs-202-02.lab
server: web, f2
Certificate chain
 0 s:CN = f2.cs-202-02.lab
   i:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   v:NotBefore: Nov 12 17:30:07 2026 GMT; NotAfter: Feb 10 17:30:07 2027 GMT
 1 s:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   i:O = OPISM Lab, CN = OPISM Stand Root CA
   v:NotBefore: Nov 12 17:30:07 2026 GMT; NotAfter: Nov 11 17:30:07 2031 GMT
---
Verify return code: 0 (ok)
=== f3.cs-202-02.lab
curl: (60) SSL certificate problem: unable to get local issuer certificate
Certificate chain
 0 s:CN = f3.cs-202-02.lab
   i:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   v:NotBefore: Nov 12 17:30:07 2026 GMT; NotAfter: Feb 10 17:30:07 2027 GMT
---
Verify return code: 21 (unable to verify the first certificate)
=== f4.cs-202-02.lab
curl: (60) SSL certificate problem: unable to get local issuer certificate
Certificate chain
 0 s:CN = f4.cs-202-02.lab
   i:O = Foreign Org, CN = Foreign Intermediate CA
   v:NotBefore: Nov 12 17:30:07 2026 GMT; NotAfter: Feb 10 17:30:07 2027 GMT
 1 s:O = Foreign Org, CN = Foreign Intermediate CA
   i:O = Foreign Org, CN = Foreign Root CA
   v:NotBefore: Nov 12 17:30:07 2026 GMT; NotAfter: Nov 11 17:30:07 2031 GMT
---
Verify return code: 20 (unable to get local issuer certificate)
=== f5.cs-202-02.lab
curl: (60) SSL certificate problem: self-signed certificate
Certificate chain
 0 s:CN = f5.cs-202-02.lab
   i:CN = f5.cs-202-02.lab
   v:NotBefore: Nov 12 17:30:07 2026 GMT; NotAfter: Feb 10 17:30:07 2027 GMT
---
Verify return code: 18 (self-signed certificate)
root@cs-202-02:/lab/work/lego/certificates# 

```

| Сервер | Помилка `curl` | `Verify return code` | Яка перевірка не пройшла | Рядок виводу, що це доводить | Що має змінити власник сервера |
|---|---|---|---|---|---|
| f1 |Невідповідність імені сертифіката|0 (ok)|	Перевірка відповідності імені сервера сертифікату|no alternative certificate subject name matches target host name 'f1.cs-202-02.lab' |Отримати сертифікат із правильним ім'ям f1.cs-202-02.lab у SAN. |
| f2 |Немає|0 (ok)|Усі перевірки пройшли успішно |server: web, f2|Нічого змінювати не потрібно. |
| f3 |Не вдалося отримати сертифікат видавця |21 (unable to verify the first certificate)|Перевірка повноти ланцюжка сертифікаті |unable to get local issuer certificateДодати проміжний сертифікат до ланцюжка, який передає сервер. |
| f4 |Не вдалося отримати сертифікат видавця |20 (unable to get local issuer certificate)|Перевірка довіри до центру сертифікації|i:O = Foreign Org, CN = Foreign Intermediate CA|Org, CN = Foreign Intermediate CA	Замінити сертифікат на виданий довіреним центром сертифікації стенда.|
| f5 |Самопідписаний сертифікат|18 (self-signed certificate) |Перевірка довіри до сертифіката|i:CN = f5.cs-202-02.lab |Замінити самопідписаний сертифікат на сертифікат від довіреного центру. |

1. Для сервера з помилкою `unable to get local issuer certificate` визначте, яка це несправність: сервер не надсилає проміжний сертифікат чи сертифікат виданий центром, якому клієнт не довіряє. Чим ці дві несправності відрізняються у виводі `openssl s_client`?
```
У моїх результатах така помилка виникла на серверах f3 і f4, але причини в них різні. Для f3 сервер не надсилає проміжний сертифікат: у ланцюжку видно тільки сертифікат самого сервера, хоча його видавцем зазначений OPISM Stand Intermediate CA. Це підтверджує Verify return code: 21 (unable to verify the first certificate). Для f4 проміжний сертифікат присутній, але він виданий іншим центром — Foreign Intermediate CA, кореневому сертифікату якого клієнт не довіряє. Це підтверджує Verify return code: 20 (unable to get local issuer certificate). Отже, у першому випадку проблема полягає у відсутності проміжного сертифіката, а в другому — у недовіреному центрі сертифікації.
```
2. Якщо серед ваших серверів є сервер з невідповідним ім'ям, чому `openssl s_client` повертає для нього `Verify return code: 0 (ok)`, а `curl` повідомляє про помилку? Якщо такого сервера немає, дайте відповідь для загального випадку.
```
У моїх результатах це сервер f1.cs-202-02.lab. Його сертифікат виданий для імені legacy.cs-202-02.lab, яке не збігається з адресою сервера. openssl s_client повертає код 0 (ok), оскільки перевірка ланцюжка сертифікатів пройшла успішно. Але curl додатково перевіряє відповідність імені сервера сертифікату. Саме тому виникає помилка no alternative certificate subject name matches target host name.
```
3. Які з п'яти серверів відповіли б на запит `curl -k`? Що при цьому втрачається порівняно з `--cacert`?
```
На запит curl -k відповіли б усі п'ять серверів, якщо вони доступні за мережею. Цей параметр вимикає перевірку сертифіката, тому запити виконуються навіть за наявності помилок довіри або невідповідності імені. На відміну від нього, --cacert дозволяє перевіряти сертифікат за вказаним кореневим центром. При використанні -k втрачається перевірка справжності сервера, тому зростає ризик підключення до підробленого або недовіреного ресурсу.
```
## Завдання 5. Ціна нового з'єднання

**Прогноз** (записати до вимірювання). Скільки RTT мине до завершення кожного етапу:

Прогноз, що для встановлення нового з'єднання TLS 1.3 потрібно приблизно 3 RTT: 1 RTT на встановлення TCP-з'єднання, 1 RTT на рукостискання TLS і 1 RTT на надсилання HTTP-запиту та отримання першої відповіді.
Для TLS 1.2 потрібно приблизно 4 RTT: 1 RTT на TCP-з'єднання, 2 RTT на рукостискання TLS і ще 1 RTT на отримання HTTP-відповіді
У разі повторного використання вже встановленого з'єднання повторно виконувати TCP- та TLS-рукостискання не потрібно, тому для отримання відповіді залишається приблизно 1 RTT. DNS-пошук може виконуватися окремо, якщо адреси ще немає в кеші.

| Етап | TLS 1.3, нове з'єднання | TLS 1.2, нове з'єднання | Повторне використання з'єднання |
|---|---|---|---|
| `time_connect` | 64.240|60.564 |0 |
| `time_appconnect` |99.318|124.873 |0 |
| `time_starttransfer` | 129.622|155.267 |31.882 |

Вивід кроків 5.2 і 5.3:
TLS 1.3
```
0.060778 0.090918 0.124200 0.155217 0.155267 2 1
0.030462 0.060580 0.093744 0.124040 0.124088 2 1
0.034099 0.064240 0.099318 0.129622 0.129684 2 1
```
TLS 1.2
```
0.030428 0.060564 0.123922 0.154234 0.154287 2 1
0.030420 0.060529 0.125597 0.155925 0.156001 2 1
0.030419 0.060586 0.124873 0.155267 0.155322 2 1
```
повторне використання з'єднання
```
0.060792 0.091042 0.124997 0.155342 0.155395 2 1
0.000018 0.000000 0.000000 0.031882 0.031933 2 0
0.000018 0.000000 0.000000 0.030303 0.030352 2 0

```
<!-- Значення в мілісекундах. Для нових з'єднань — медіана трьох вимірювань. -->

| Запит | `namelookup` | `connect` | `appconnect` | `starttransfer` | `total` | `num_connects` |
|---|---|---|---|---|---|---|
| TLS 1.3, нове |34.099 |64.240 |99.318 |129.622 |129.684 |1 |
| TLS 1.2, нове |30.420|60.564 |124.873 |155.267 |155.322 |1 |
| 5.3, перший запит | 60.792|91.042 |124.997 |155.342 |155.395 |1 |
| 5.3, другий запит | 0.018| 0|0 |31.882 |31.933 |0 |
| 5.3, третій запит | 0.018|0 |0 |30.303 |30.352 |0 |

RTT, визначений вимірюванням, і з якого значення його визначено:

RTT, визначений вимірюванням, становить приблизно 30 мс. Його визначено за різницею між `time_connect` і `time_namelookup` для нових з'єднань TLS 1.3 та TLS 1.2:

- TLS 1.3: 64,240 − 34,099 = 30,141 мс.
- TLS 1.2: 60,564 − 30,420 = 30,144 мс.

1. Виразіть кожне виміряне значення через RTT. Де вимірювання відрізняється від цілого числа RTT і чим це пояснюється?
```
Один RTT за моїми вимірюваннями становить приблизно 30 мс. Для TLS 1.3 значення time_connect, time_appconnect і time_starttransfer становлять приблизно 2,1; 3,3 і 4,3 RTT. Для TLS 1.2 — приблизно 2; 4,2 і 5,2 RTT. При повторному використанні з'єднання time_starttransfer становить близько 1 RTT. Відхилення пояснюються затримками обробки запитів і коливаннями часу передавання даних мережею.
```
2. Чому `time_namelookup` не близький до нуля, хоча резолвер уже має ім'я в кеші? За яких умов він був би приблизно вдвічі більшим?
```
Навіть якщо DNS-запис уже є в кеші, клієнту потрібно надіслати запит резолверу й отримати відповідь. Тому time_namelookup у моїх результатах становить приблизно 30–34 мс. Якщо потрібного запису не буде в кеші, резолверу доведеться додатково звертатися до інших DNS-серверів. Через це час пошуку може збільшитися приблизно вдвічі.
```
3. Сервіс виконує 200 послідовних запитів до цього сервера. Розрахуйте, скільки часу займе лише очікування мережі, якщо кожен запит встановлює нове з'єднання TLS 1.3, нове з'єднання TLS 1.2 і якщо всі запити йдуть через одне з'єднання.
```
Для розрахунку використаємо time_starttransfer. Для TLS 1.3: 129,622 × 200 = 25 924,4 мс, тобто приблизно 25,9 секунди. Для TLS 1.2: 155,267 × 200 = 31 053,4 мс, тобто приблизно 31,1 секунди. При повторному використанні одного з'єднання перший запит займає 155,342 мс, а наступні — у середньому близько 31,1 мс. Загалом це приблизно 6,35 секунди. Отже, повторне використання з'єднання значно скорочує час очікування.
```
## Висновки

Що виявилося несподіваним. Несподіваним для мене стало те, що TLS 1.2 і TLS 1.3, хоча обидва забезпечують захищене з'єднання, мають різний процес рукостискання. Це було видно з моїх виводів: у TLS 1.3 рукостискання потребувало менше RTT, ніж у TLS 1.2. Також відрізнялася видимість сертифікатів у мережевому дампі.

Яке питання залишилося без відповіді. Після виконання лабораторної мені залишилося не до кінця зрозумілим, чому саме TLS 1.3 працює швидше за TLS 1.2. У виводах я побачила різницю в кількості RTT та повідомленнях під час рукостискання, але хотілося б краще зрозуміти, як саме ці зміни впливають на швидкість встановлення з'єднання.

## Декларація використання ШІ

Використовувалися інструменти ШІ:  так

<!-- Якщо так, заповнити таблицю: один рядок на кожне звернення, що вплинуло на звіт. Промпт наводити дослівно. -->

| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 |GPT-6|сформуй думку чітко та структуровано(мій текст)завдання 4 та 5 |Сформував думку чіткою без зайвого  |
| 2 |GPT-6|Пояснюй крок за кроком, я вперше це роблю|надав покрокову відповідь з чітким виконанням завдання |
| 3 |GPT-6|розкажи що відбулось простими словами|пояснив |


