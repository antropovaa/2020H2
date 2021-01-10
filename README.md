## Лабораторная работа 2 – NTP-клиент

### Инструкция по запуску
Запускается из командной строки с необязательными параметрами:  
`python3 ntp_client.py -s <server> -t <timeout>`  
Значения по умолчанию:
* сервер: [pool.ntp.org](https://www.ntppool.org/ru/)
* порт: 123
* время ожидания ответа от сервера: 10 сек


### Тестирование работы
Пример работы клиента:
```
$ python3 ntp_client.py                     
Server: pool.ntp.org
--------------------------------------
Server time: 2021-01-10 22:57:41.563910
Local time: 2021-01-10 22:57:41.369149
Round trip time: 92 ms
Offset: 149 ms
```

Превышено время запроса:
```
$ python3 ntp_client.py -s ntp.nict.jp
Server: ntp.nict.jp
--------------------------------------
Timeout! Try to connect to the server again
```

### Описание протокола
**NTP (Network Time Protocol)** — протокол, который работает поверх UDP и используется для синхронизации локальных часов с часами на сервере точного времени. Актуальная версия – 4, она же используется в лабораторной работе.

#### Формат пакетов

       0                   1                   2                   3
       0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |LI | VN  |Mode |    Stratum     |     Poll      |  Precision   |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                         Root Delay                            |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                         Root Dispersion                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                          Reference ID                         |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      +                     Reference Timestamp (64)                  +
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      +                      Origin Timestamp (64)                    +
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      +                      Receive Timestamp (64)                   +
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      +                      Transmit Timestamp (64)                  +
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      .                                                               .
      .                    Extension Field 1 (variable)               .
      .                                                               .
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      .                                                               .
      .                    Extension Field 2 (variable)               .
      .                                                               .
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                          Key Identifier                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      |                            dgst (128)                         |
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

* **Leap indicator (LI), 2 бита** — число, предупреждающее о секунде координации. Может быть от 0 до 3, где 0 — нет коррекции, 1 — последняя минута дня содержит 61 с, 2 — последняя минута дня содержит 59 с, 3 — неисправность сервера. При значении 3 полученным данным доверять не следует. Вместо этого нужно обратиться к другому серверу. Наш псевдосервер будет всегда возвращать 0.
* **Version number (VN), 3 бита** — номер версии протокола NTP (1–4). Мы поставим туда 3.
* **Mode, 3 бита** — режим работы отправителя пакета. Значение от 0 до 7, где 3 — клиент, а 4 — сервер.
* **Stratum, 1 байт** — сколько посредников между клиентом и эталонными часами (включая сам NTP-сервер). 1 — сервер берет данные непосредственно с атомных (или других точных) часов, то есть между клиентом и часами только один посредник (сам сервер); 2 — сервер берет данные с сервера со значением Stratum 1 и так далее.
* **Poll, 1 байт** — целое число, задающее интервал в секундах между последовательными обращениями. Клиент может указать здесь интервал, с которым он хочет отправлять запросы на сервер, а сервер — интервал, с которым он разрешает, чтобы его опрашивали.
* **Precision (точность), 1 байт** — число, которое сообщает точность локальных системных часов. Значение равно двоичному логарифму секунд.
* **Root delay (задержка сервера), 4 байта** — время, за которое показания эталонных часов доходят до сервера NTP. Задается как число секунд с фиксированной запятой.
* **Root dispersion, 4 байта** — разброс показаний сервера.
* **RefID, 4 байта** (идентификатор источника) — ID часов. Если поле Stratum равно единице, то RefID — имя атомных часов (четыре символа ASCII). Если текущий сервер NTP использует показания другого сервера, то в RefID записан IP-адрес этого сервера.
* **Reference, 8 байт** — последние показания часов сервера.
* **Originate, 8 байт** — время, когда пакет был отправлен, по версии сервера.
* **Receive, 8 байт** — время получения запроса сервером.
* **Transmit, 8 байт** — время отправки ответа сервера клиенту, которое заполняет клиент.

Со стороны клиента заполняются только поля LI (unknown), VN (4), Mode (3) и Transmit.


#### Процесс передачи данных
Клиент посылает запрос на сервер, запоминая, когда этот запрос был отправлен. Сервер принимает пакет, запоминает и записывает в пакет время приема, заполняет время отправки и отвечает клиенту. Клиент запоминает, когда он получил ответ, и получает нечто вроде RTT (Round-Trip Time) до сервера. Дальше он определяет, сколько времени понадобилось пакету, чтобы дойти от сервера обратно ему (время между запросом и ответом клиента минус время обработки пакета на сервере, деленное на два).