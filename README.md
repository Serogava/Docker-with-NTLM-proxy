
# Настройка Debian на работу с proxy

### Настраиваем loopback интерфейс
Добавляем в /etc/network/interface строки с настройками дополнительного интефейса loopback
```
auto lo:10
iface lo:10 inet static
        address <IP>
        netmask <MASK>
```
Где:  
- lo:10 - имя loopback интерфейса, номер интерфейсча может быть любым; 
- IP - адрес интерфейса, далее будет использоваться в конфигурациях proxy
Перезапускаем сервис сети
```
sudo systemctl restart networking
```

#### Устанавливаем и настраиваем cntlm
Обновляем пакеты
```
sudo apt update &&  sudo apt upgrade
```
Устанавливаем пакет cntlm
```
sudo apt install cntlm -y
```
Генерируем хэш пароля для пользователя $USER
```
sudo cntlm -H -d DOMAIN -u $USER
```
Где:
- *DOMAIN* — это используемый домен
- *$USER* — пользователь Windows.

Полученный результат необходимо записать, он пригодиться далее.

Редактируем */etc/cntlm.conf*. В этом файле вы найдете четыре строки, которые необходимо настроить:
```
Username <MS_USERNAME>
Domain <DOMAIN>
Password <HASH>
PassLM <HASH>
PassNT <HASH>
PassNTLMv2 <HASH>
```

Где:
- *MS_USERNAME* — это ваше действительное имя пользователя Windows.
- *DOMAIN* — это ваш домен Windows.
- *HASH* — Полученные после выполнения команды *sudo cntlm -H -d DOMAIN -u $USER*

Заполняем поля *Proxy*, где прописывается proxy сервер сети и его порт. Например:
```
Proxy 192.168.1.10:8080
Proxy 192.168.1.11:8080
```
В поле *Listenen* прописывается адрес loopback, который был поднят ранее, и порт на котором будет работать локальный proxy
```
Listen          <IP>:<PORT>
```
Выполняем преезагрузку сервиса cntlm
```
sudo systemctl restart cntlm
```

#### Настройка docker на работу с proxy
Для настройки HTTP_PROXYHTTPS_PROXYNO_PROXY необходимо выполнить коррективы в сервисе docker.service

Создать директорию
```
sudo mkdir -p /etc/systemd/system/docker.service.d
```
Создать файл */etc/systemd/system/docker.service.d/https-proxy.conf* и вписать в него настройки proxy
```
[Service]
Environment="HTTP_PROXY=http://<IP:PORT>/" "HTTPS_PROXY=http://<IP:PORT>/" "NO_PROXY=localhost,127.0.0.1,<other>"
```
Где
- HTTP_PROXY - адрес *Proxy* для протокола HTTP
- HTTPS_PROXY - адрес *Proxy* для протокола HTTPS 
- NO_PROXY - перечисление адресов куда необходимо ходить без proxy
- IP:PORT - адрес и порт указанные в директиве *Listenen* файла /etc/cntlm.conf
Применяем изменения
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```


