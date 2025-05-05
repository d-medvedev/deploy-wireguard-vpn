# Инструкция по установке WireGuard в Docker-контейнере

## Подготовка

1. **Установите Docker (если еще не установлен):**
   ```bash
   sudo apt update
   sudo apt install docker.io docker-compose
   sudo systemctl enable --now docker
   ```

2. **Создайте папку для конфигурации:**
   ```bash
   mkdir -p ~/wireguard/config
   cd ~/wireguard
   ```

## Настройка с использованием Docker Compose

3. **Создайте docker-compose.yml:**
   ```bash
   nano docker-compose.yml
   ```

4. **Добавьте следующую конфигурацию:**
   ```yaml
   services:
     wireguard:
       image: linuxserver/wireguard
       container_name: wireguard
       cap_add:
         - NET_ADMIN
         - SYS_MODULE
       environment:
         - PUID=1000
         - PGID=1000
         - TZ=Europe/Moscow
         - SERVERURL=auto
         - SERVERPORT=51820
         - PEERS=3
         - PEERDNS=auto
         - INTERNAL_SUBNET=10.13.13.0
       volumes:
         - ./config:/config
         - /lib/modules:/lib/modules
       ports:
         - 51820:51820/udp
       restart: unless-stopped
       sysctls:
         - net.ipv4.conf.all.src_valid_mark=1
   ```

   > Примечания:
   > - `SERVERURL=auto` - определит внешний IP автоматически
   > - `PEERS=3` - создаст 3 клиентских профиля
   > - Измените TZ на ваш часовой пояс

5. **Запустите контейнер:**
   ```bash
   docker compose up -d
   ```

6. **Проверьте статус:**
   ```bash
   docker ps
   ```

## Настройка брандмауэра

7. **Откройте порт WireGuard:**
   ```bash
   sudo ufw allow 51820/udp
   ```

## Получение клиентских конфигураций

8. **Клиентские конфигурации находятся в папке config:**
   ```bash
   ls -la ~/wireguard/config/peer*
   ```

9. **Получите QR-код для мобильных устройств:**
   ```bash
   docker exec -it wireguard /app/show-peer 1
   ```
   Команда выведет QR-код для первого пира. Замените `1` на номер нужного пира.

## Обслуживание

- **Перезапуск контейнера:**
  ```bash
  cd ~/wireguard
  docker-compose restart
  ```

- **Остановка сервера:**
  ```bash
  docker-compose down
  ```

- **Добавление новых клиентов:**
  Измените переменную `PEERS` в файле docker-compose.yml и перезапустите контейнер.

## Примечания
- SSH-соединение не будет затронуто
- Контейнер настроен на автозапуск при перезагрузке сервера
- Все клиентские конфигурации сохраняются в папке config
