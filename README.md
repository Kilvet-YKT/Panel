# WEB панель для AmneziaWG
WEB панель для управления VPN серверами Amnezia AWG
## Установка проводилась на сервер Ubuntu Server 24.04
```bash
sudo apt update && apt upgrade -y

sudo apt install -y curl docker.io docker-compose-v2

git clone https://github.com/Kilvet-YKT/Panel.git

cd Panel/
```
## Отредактируйте файл docker-compose.yml
Обязательно найти поменять в данные в  этих строках на свои

      - NGINX_USER=admin          # Логин для базовой аутентификации (ОБЯЗАТЕЛЬНО ИЗМЕНИТЕ!)
      
      - NGINX_PASSWORD=P@ssw0rd # Пароль для аутентификации (ОБЯЗАТЕЛЬНО ИЗМЕНИТЕ!)`:

Содержимое файла docker-compose.yml
```
version: '3.8'

services:
  amnezia-web-ui:
    image: alexishw/amneziawg-web-ui:latest
    container_name: amnezia-web-ui
    restart: unless-stopped
    ports:
      # Левый порт — на хосте, правый — внутри контейнера.
      # Порт веб-интерфейса. NGINX_PORT внутри контейнера должен совпадать с правым портом здесь.
      - "8080:8080/tcp"
      # Пример порта для VPN. Оставьте закомментированным, если пока не знаете, какой порт будете использ>
      - "51820:51820/udp"
    environment:
      # --- Обязательные переменные ---
      - NGINX_PORT=8080           # Порт веб-интерфейса внутри контейнера. Должен совпадать с правым порт>
      - NGINX_USER=admin          # Логин для базовой аутентификации (ОБЯЗАТЕЛЬНО ИЗМЕНИТЕ!)
      - NGINX_PASSWORD=P@ssw0rd # Пароль для аутентификации (ОБЯЗАТЕЛЬНО ИЗМЕНИТЕ!)

      # --- Опциональные, но рекомендуемые ---
      - AUTO_START_SERVERS=true   # Автоматически запускать VPN-серверы при старте контейнера
      - DEFAULT_MTU=1420          # MTU по умолчанию для новых серверов
      - DEFAULT_SUBNET=10.0.0.0/24 # Сеть по умолчанию для новых серверов
      - DEFAULT_PORT=51820        # Порт по умолчанию для новых серверов
      - DEFAULT_DNS=8.8.8.8,1.1.1.1 # DNS-серверы по умолчанию для клиентов

      # --- Переменные для SSL (опционально) ---
      # - SSL_EMAIL=your-email@example.com
      # - SSL_DOMAIN=vpn.your-domain.com

      # --- Защита по IP (опционально) ---
      # - IP_LIST="YOUR_HOME_IP, 192.168.1.0/24"
    volumes:
      # Основное хранилище конфигураций AmneziaWG
      - amnezia-data:/etc/amnezia
      # Если используете SSL, раскомментируйте следующую строку для сохранения сертификатов
      # - ssl-data:/etc/letsencrypt
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    devices:
      - /dev/net/tun
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1
      - net.ipv6.conf.all.disable_ipv6=0
      - net.ipv6.conf.all.forwarding=1
      - net.ipv6.conf.default.forwarding=1

volumes:
  amnezia-data:
  # ssl-data: # Раскомментируйте, если используете SSL


```
Сохранить изменения и закрыть файл после чего запустить Docker
```bash
docker compose up -d
```
После запуска контейнера WEB панель будет доступна по адресу **http://<Ваш_IP_адрес(домен)>:8080**
