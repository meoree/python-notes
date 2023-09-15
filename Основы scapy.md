# Основы scapy

Импорт:
```
from scapy.all import *
from scapy.layers.inet import IP, UDP
from scapy.layers.l2 import Ether, MPacketPreamble
```

### Создание пакета:
`b = IP(ttl=10)`

У объекта b есть разные методы, например:
`b.src`
```
In [16]: b.ttl
Out[16]: 10
```
Просмотр пакета:
1. Метод show()
```
In [14]: b.show()
###[ IP ]### 
  version   = 4
  ihl       = None
  tos       = 0x0
  len       = None
  id        = 1
  flags     = 
  frag      = 0
  ttl       = 10
  proto     = ip
  chksum    = None
  src       = 127.0.0.1
  dst       = 127.0.0.1
  \options   \
```
2. Метод show2()
```
In [20]: b.show2()
###[ IP ]### 
  version   = 4
  ihl       = 5
  tos       = 0x0
  len       = 20
  id        = 1
  flags     = 
  frag      = 0
  ttl       = 10
  proto     = ip
  chksum    = 0xb2e7
  src       = 127.0.0.1
  dst       = 127.0.0.1
```

### Stacking layers 
Для соединения уровней пакета используется оператор /. 
```
In [22]: a = IP() / UDP()
In [23]: a
Out[23]: <IP  frag=0 proto=udp |<UDP  |>>
```

Конвертирование:
```
In [33]: a = raw(b)
In [34]: b
Out[34]: <IP  ttl=10 |>
In [35]: a
Out[35]: b'E\x00\x00\x14\x00\x01\x00\x00\n\x00\xb2\xe7\x7f\x00\x00\x01\x7f\x00\x00\x01'
In [36]: a1 = Ether(a)
In [37]: a1
Out[37]: <Ether  dst=45:00:00:14:00:01 src=00:00:0a:00:b2:e7 type=0x7f00 |<Raw  load='\x00\x01\x7f\x00\x00\x01' |>>
```

### Создание нескольких пакетов
В квадратных скобках перечисляются конкретные значения через запятую, которые нужно сформировать. А в круглых скобках написан интервал, от какого и до какого числа нужно сформировать значения. 
```
In [40]: a = IP(dst="192.168.89.102/24")

In [41]: a
Out[41]: <IP  dst=Net("192.168.89.102/24") |>

In [42]: b = [ip for ip in a]

In [43]: b
Out[43]: 
[<IP  dst=192.168.89.0 |>,
 <IP  dst=192.168.89.1 |>,
 <IP  dst=192.168.89.2 |>,
 <IP  dst=192.168.89.3 |>,
...
 <IP  dst=192.168.89.255 |>]
```
b - это список пакетов:
```
In [44]: c = b[0]
In [45]: c
Out[45]: <IP  dst=192.168.89.0 |>
In [46]: c.show()
###[ IP ]### 
  version   = 4
  ihl       = None
  tos       = 0x0
  len       = None
  id        = 1
  flags     = 
  frag      = 0
  ttl       = 64
  proto     = ip
  chksum    = None
  src       = 10.120.254.40
  dst       = 192.168.89.0
  \options   \
```
Соединение двух пакетов в генераторе:
```
In [48]: c = TCP(dport=[10, 14])
In [49]: c
Out[49]: <TCP  dport=['10', '14'] |>
In [50]: [p for p in a / c]
Out[50]: 
[<IP  frag=0 proto=tcp dst=192.168.89.0 |<TCP  dport=10 |>>,
 <IP  frag=0 proto=tcp dst=192.168.89.0 |<TCP  dport=14 |>>,
 <IP  frag=0 proto=tcp dst=192.168.89.1 |<TCP  dport=10 |>>,
 <IP  frag=0 proto=tcp dst=192.168.89.1 |<TCP  dport=14 |>>,
...
```

**Объект PacketList** 

Хранит список пакетов. 
```
In [59]: pack_1 = Ether(dst="00:11:AA:BB:CC:DD")
In [60]: pack_1
Out[60]: <Ether  dst=00:11:AA:BB:CC:DD |>

In [63]: pack_2 = IP(ttl=[1, 3, (5, 8), 10])
In [64]: pack_2
Out[64]: <IP  ttl=[1, 3, (5, 8), 10] |>

In [65]: result = PacketList([p for p in pack_1 / pack_2])

In [66]: result
Out[66]: <PacketList: TCP:0 UDP:0 ICMP:0 Other:7>

In [67]: result.show()
0000 Ether / 127.0.0.1 > 127.0.0.1 ip
0001 Ether / 127.0.0.1 > 127.0.0.1 ip
0002 Ether / 127.0.0.1 > 127.0.0.1 ip
0003 Ether / 127.0.0.1 > 127.0.0.1 ip
0004 Ether / 127.0.0.1 > 127.0.0.1 ip
0005 Ether / 127.0.0.1 > 127.0.0.1 ip
0006 Ether / 127.0.0.1 > 127.0.0.1 ip

In [69]: packet0 = result[0]

In [70]: packet0.show()
###[ Ethernet ]### 
  dst       = 00:11:AA:BB:CC:DD
  src       = 00:00:00:00:00:00
  type      = IPv4
###[ IP ]### 
     version   = 4
     ihl       = None
     tos       = 0x0
     len       = None
     id        = 1
     flags     = 
     frag      = 0
     ttl       = 1
     proto     = ip
     chksum    = None
     src       = 127.0.0.1
     dst       = 127.0.0.1
     \options   \
In [72]: packet1.show()
###[ Ethernet ]### 
  dst       = 00:11:AA:BB:CC:DD
  src       = 00:00:00:00:00:00
  type      = IPv4
###[ IP ]### 
     version   = 4
     ihl       = None
     tos       = 0x0
     len       = None
     id        = 1
     flags     = 
     frag      = 0
     ttl       = 3
     proto     = ip
     chksum    = None
     src       = 127.0.0.1
     dst       = 127.0.0.1
     \options   \
```

### Отправка пакетов
Функция send() и sendp(). 
send() - отправляет пакеты с уровнем 3. 
sendp() - отправляет пакеты с уровнем 2. 
Если пакет имеет Ether(), то его нужно отправить на уровне 2. Если не имеет, то на уровне 3. 
```
In [73]: send(packet1)
.
Sent 1 packets.

In [75]: sendp(packet1)
.
Sent 1 packets.

In [76]: send(result)
.......
Sent 7 packets.
```
Отправленные пакеты можно добавлять в PacketList с помощью параметра return_packets=True. 
```
In [86]: return_packs = send(IP(dst="192.168.89.102", ttl=100) / ICMP(), return_packets=True)
.
Sent 1 packets.
In [87]: return_packs.show()
0000 IP / ICMP 10.120.254.40 > 192.168.89.102 echo-request 0
In [88]: return_packs[0].show()
###[ IP ]### 
  version   = 4
  ihl       = None
  tos       = 0x0
  len       = None
  id        = 1
  flags     = 
  frag      = 0
  ttl       = 100
  proto     = icmp
  chksum    = None
  src       = 10.120.254.40
  dst       = 192.168.89.102
  \options   \
###[ ICMP ]### 
     type      = echo-request
     code      = 0
     chksum    = None
     id        = 0x0
     seq       = 0x0
     unused    = ''
```
Зацикливание пакета:
```
# Интервал по умолчанию. 
In [90]: sendp(packet1, loop=1)
.........................................................................................................................^C
Sent 3733 packets.

# Интервал одна секунда, бесконечно
In [92]: sendp(packet1, loop=1, inter=1)
....^C
Sent 4 packets.

# Интервал 0.5 секунды, количество 10
In [94]: sendp(packet1, loop=1, inter=0.5, count=10)
..........
Sent 10 packets.
```

### Функция fuzz()
Используется чтобы изменить любое дефолтное значение, кроме тех, что вычисляются (например Checksum). 
Подставляются рандомные значения для полей, или можно выбрать точно фиксированное значение. 
Для уровня IP() fuzz() не подставляет рандомные IP адреса. Для этого использовать функцию RandIP().
```
In [5]: b = send(IP(dst="ya.ru") / fuzz(UDP() / NTP()), return_packe
   ...: ts=True)
.
Sent 1 packets.

In [7]: b[0].show()
###[ IP ]### 
  version   = 4
  ihl       = None
  tos       = 0x0
  len       = None
  id        = 1
  flags     = 
  frag      = 0
  ttl       = 64
  proto     = udp
  chksum    = None
  src       = 10.120.254.40
  dst       = 87.250.250.242
  \options   \
###[ UDP ]### 
     sport     = ntp
     dport     = ntp
     len       = None
     chksum    = None
###[ NTPHeader ]### 
        leap      = last minute of the day has 59 seconds
        version   = 6
        mode      = broadcast
        stratum   = 218
        poll      = 94
        precision = 98
        delay     = 49703.2568
        dispersion= 30351.3253
        id        = 226.216.178.96
        ref       = 1520021542.87616558
        orig      = --
        recv      = 950382826.205300524
        sent      = --
# Формируются пакеты только 4 версии ntp
In [8]: b = send(IP(dst="ya.ru") / fuzz(UDP() / NTP(version=4)), ret
   ...: urn_packets=True)
.
Sent 1 packets.
```



## Отправить и принять пакеты 
1. sr() - отправляет и принимает пакеты обратно. Возвращает пару пакет и ответ, и пакеты, на которых нет ответа. 
2. sr1() - возвращает только один ответ. Пакеты должны быть уровня 3. 
3. srp() - тоже самое, что и sr1() только на втором уровне. 

Если ответа нет, то None. 



