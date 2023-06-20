# Модули pexpect, telnetlib, netmiko
#python/Подключение_к_оборудованию  
Все модули для подключения по ssh и telnet. 
## Хранение паролей
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-25%20%D0%B2%2021.26.02.jpg)
## Модуль Pexpect
`pip install pexpect`
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-25%20%D0%B2%2021.33.08.jpg)
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-25%20%D0%B2%2021.33.36.jpg)
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-25%20%D0%B2%2021.38.03.jpg)
Цикл работы:
1) `r = expect.spawn(‘ssh user@ip»)`
Подключаемся к оборудованию
2) Ожидаем пароль
`r.expect("Password")`
3) Отправляем пароль
`r.sendline("password")`
4) Ожидаем приглашение
`r.expect('R1[>#]")`
Непривилигированный режим циско. 
Expect - регулярное выражение. Он ищет до первого приглашения. Если несколько команд подряд, то он прочитает только первую. 
5) Отправляем команду
`r.sendline('command')`
6) Ожидаем приглашение
`r.expect('R1>")`
7) Считать вывод команды, читаем до приглашения
`r.before`

**Отличие sendline от send:**
Sendline добавляет перевод строки, send не добавляет. 

`r.expect(['R2', 'R1'])` - можно перечислить, возвратит индекс того, что совпало. 
**Перехват исключения:**
1)![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-25%20%D0%B2%2021.50.12.jpg)
Индекс 2 можно его отлавливать. Исключение не вываливается. 
2)
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-25%20%D0%B2%2022.29.32.jpg)

**Обработка вывода**
Строка возвращается байтовая. Варианты:
1) `r.before.decode`
2) Декодировать строку после получения вывода. 

Часто оборудование возвращает \r\n. Можно заменить \r\n на \n с помощью replace(). Чтобы в дальнейшем было удобнее обрабатывать строку регулярным выражением. 
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-25%20%D0%B2%2022.07.45.jpg)

Подключаться к оборудованию нужно с помощью ::менеджера контекста.:: Он автоматически закроет сессию. 
::Пейджинг:: - когда команда отображается не полностью, а кусками которые нужно пролистывать. Для каких-то супер огромных команд его можно не отключать, потому может не все поместиться в буфер. Но для обычных команд show его нужно отключить. 
На циске отключить пейджинг: `ssh.sendline("terminal lenth 0')`

Последовательное подключение к оборудованию:
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-25%20%D0%B2%2022.09.29.jpg)

Промт:
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-25%20%D0%B2%2022.34.21.jpg)

Если несколько команд:
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-25%20%D0%B2%2022.44.58.jpg)

::Если paging отключен:::
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-26%20%D0%B2%2023.28.07.jpg)
Плюс в выводе появляются символы пробела, которые нужно удалить:
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-26%20%D0%B2%2023.35.51.jpg)

## Модуль telnetlib
Принцип такой же, только всегда отправляем байковую строку. 
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-26%20%D0%B2%2023.42.32.jpg)

## Модуль Paramiko
Только SSHv2. Ориентирован на сетевое оборудование. Поддерживает функционал клиента и сервера. 
`pip install paramiko`
Всегда надо ждать - sleep. 
Если подключаемся к оборудованию впервые:
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-28%20%D0%B2%2022.20.28.jpg)
Отключаем аутентификацию по ключам, так как в сетевом оборудовании ее нет. 
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-28%20%D0%B2%2022.23.14.jpg)


Стадия подключения:
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-28%20%D0%B2%2022.23.55.jpg)
Открываем сессию, в которой будем вводить команды и считывать результаты:
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-28%20%D0%B2%2022.41.40.jpg)
В данном модуле rcv можно использовать не парами запрос-ответ, а в конце, после ввода всех команд. Первый rcv здесь нужен, чтобы не считывать вывод захода в enable режим. 

С исключением:
Надо импортировать модуль сокет:
`import socket`![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-28%20%D0%B2%2022.45.57.jpg)
Обработка всех исключений paramiko:
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-28%20%D0%B2%2022.54.37.jpg)

 Пример с умным чтением строки:
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-10-28%20%D0%B2%2023.03.22.jpg)

## Модуль Netmiko
`pip install netmiko`
Позволяется работать с сетевым оборудованием. С этим модулей надо писать меньше куда. 
Поддерживает не все платформы. 
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-11-02%20%D0%B2%2021.46.33.jpg)

![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-11-02%20%D0%B2%2021.47.00.jpg)
send_command - для команд типа show, display и тд
send_config_set - одна или несколько конфигурационных команд
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-11-02%20%D0%B2%2021.49.32.jpg)
Методы:
Зайти в конфиг:
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-11-02%20%D0%B2%2021.51.12.jpg)
Отправить команду и не выходить из конфига:
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-11-02%20%D0%B2%2021.51.30.jpg)
Выйти из конфига:
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-11-02%20%D0%B2%2021.52.03.jpg)
Найди промт:
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-11-02%20%D0%B2%2021.52.17.jpg)

::Netmiko автоматически отключает paging.::

Пример:
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-11-02%20%D0%B2%2022.04.46.jpg) 

Пример с импортов конфигураций из файла yaml:
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-11-02%20%D0%B2%2022.07.00.jpg)

Фича проверки есть ли ошибка в веденной команде, процесс останавливается. 
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-11-02%20%D0%B2%2022.26.37.jpg)

 Если paging включен:
write_channel не ждет приглашения.  
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-11-02%20%D0%B2%2022.30.51.jpg)

Парсинг вывода:
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-11-02%20%D0%B2%2022.35.47.jpg)

send_config_from_file
В файле текстовом указаны команды для конфига:
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-11-08%20%D0%B2%2015.38.08.jpg)
Передача команд из файла оборудованию:
![](%D0%9C%D0%BE%D0%B4%D1%83%D0%BB%D0%B8%20pexpect,%20telnetlib,%20netmiko/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202022-11-08%20%D0%B2%2015.39.08.jpg)


Обработка исключений:
```
    #Срабатывание исключений в случае неудачного создания соединения    
    except (AuthenticationException):
        print ("Authentication failure: " + ip_address_of_device)
        continue
    except (NetMikoTimeoutException):
        print ("Timeout to device: " + ip_address_of_device)
        continue
    except (EOFError):
        print ("End of file while attempting device " + ip_address_of_device)
        continue
    except (SSHException):
        print ("SSH Issue. Are you sure SSH is enabled? " + ip_address_of_device)
        continue
    except Exception as unknown_error:
        print ("Some other error: " + str(unknown_error))
        continue
    output = net_connect.send_config_set(commands_list)
    print (output)
```




