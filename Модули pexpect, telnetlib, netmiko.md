# Модули pexpect, telnetlib, netmiko

Эти модули используются для подключения по ssh и telnet. 

## Модуль Pexpect
`pip install pexpect`

![Снимок экрана 2022-10-25 в 21 38 03](https://github.com/meoree/python-notes/assets/31919838/6cf2dda1-db40-46dc-8833-1b6813120ef6)

Цикл работы:
1) Подключаемся к оборудованию
`r = expect.spawn(‘ssh user@ip»)`
2) Ожидаем пароль
`r.expect("Password")`
3) Отправляем пароль
`r.sendline("password")`
4) Ожидаем приглашение
`r.expect('R1[>#]")`

Попадаем в непривилигированный режим Cisco. 
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


**Обработка вывода**

Строка возвращается байтовая. 
Варианты:
1) `r.before.decode`
2) Декодировать строку после получения вывода. 

Часто оборудование возвращает \r\n. Можно заменить \r\n на \n с помощью replace(). Чтобы в дальнейшем было удобнее обрабатывать строку регулярным выражением. 

Подключаться к оборудованию нужно с помощью менеджера контекста.Он автоматически закроет сессию. 

**Paging** - когда команда отображается не полностью, а кусками которые нужно пролистывать. Для каких-то супер огромных команд его можно не отключать, потому может не все поместиться в буфер. Но для обычных команд show его нужно отключить. 
На Cisco отключить пейджинг: `ssh.sendline("terminal lenth 0')`


## Модуль telnetlib
Принцип такой же как в pexpect, только всегда отправляем байковую строку. 

![Снимок экрана 2022-10-26 в 23 42 32](https://github.com/meoree/python-notes/assets/31919838/0f2ce95b-9c56-4980-9658-ac1cdbde5e4d)


## Модуль Paramiko
Только SSHv2. Ориентирован на сетевое оборудование. Поддерживает функционал клиента и сервера. 
`pip install paramiko`
Всегда надо ждать - sleep. 
Если подключаемся к оборудованию впервые:
`cl.set_missing_host_key_policy(paramiko.AutoAddPolicy())`

Если подключаемся к сетевому оборудованию, где нет аутентификации по ключам: 
```
look_for_keys=False,
allow_agent=False
```


Стадия подключения:
![Снимок экрана 2022-10-28 в 22 23 55](https://github.com/meoree/python-notes/assets/31919838/faf46ceb-c43b-4d09-a39b-ef521e932462)

Открываем сессию, в которой будем вводить команды и считывать результаты:
![Снимок экрана 2022-10-28 в 22 41 40](https://github.com/meoree/python-notes/assets/31919838/821d5c15-b38b-4657-a6f2-f73f03de3907)

В данном модуле rcv можно использовать не парами запрос-ответ, а в конце, после ввода всех команд. Первый rcv здесь нужен, чтобы не считывать вывод захода в enable режим. 

**Обработка исключений:**

Надо импортировать модуль сокет: `import socket`

`expect socket.timeout:`

Обработка всех исключений paramiko: `except paramiko.SSHException as error:`

## Модуль Netmiko

`pip install netmiko`

Позволяет работать с сетевым оборудованием. С этим модулем кода надо писать намного меньше, но поддерживает не все платформы. Netmiko автоматически отключает paging.

**Отправка команд:**

send_command - для команд типа show, display и тд
send_config_set - одна или несколько конфигурационных команд

Передача команд из файла оборудованию:
`ssh.send_config_from_file("cfg_commands.txt")`

**Методы:**

Зайти в конфигурационный режим:
`ssh.config_mode()`

Отправить команду и не выходить из конфига:
`ssh.send_config_set("loging 10.10.10.1", exit_config_mode=False)`

Выйти из конфига:
`ssh.exit_config_mode()`

Найти промт:
`ssh.fing_promt()`


**Обработка исключений:**
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




