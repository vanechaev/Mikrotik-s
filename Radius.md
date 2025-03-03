# Настройка Wi-Fi аутентификации через RADIUS на MikroTik

## 1. Настройка RADIUS-сервера

### 1.1. Установка и настройка NPS (Network Policy Server) на Windows Server
1. **Установите NPS**:
   - Откройте "Диспетчер серверов" → "Добавить роли и компоненты".
   - Выберите "Службы политики и доступа к сети" → "Сервер политики сети (NPS)".
   - Завершите установку.
2. **Добавьте MikroTik в список доверенных клиентов RADIUS**:
   - Откройте "NPS" → "RADIUS-клиенты" → "Создать новый".
   - Укажите имя (например, "MikroTik AP"), IP-адрес MikroTik и общий секретный ключ.
3. **Создайте политику аутентификации Wi-Fi пользователей**:
   - Перейдите в "Политики сети" → "Создать новую".
   - Добавьте условие "Группа пользователей" (например, `WiFi-Users` в AD).
   - Включите метод аутентификации **PEAP/MSCHAPv2**.
   - Настройте политики ограничений (например, доступ только с определенных VLAN).
4. **Примените изменения и перезапустите службу NPS**:
   ```powershell
   net stop npas
   net start npas
   ```

### 1.2. Альтернативная настройка FreeRADIUS в Linux
1. **Установите FreeRADIUS** (Ubuntu/Debian):
   ```bash
   sudo apt update && sudo apt install freeradius -y
   ```
2. **Настройте клиент RADIUS для MikroTik**:
   - В файле `/etc/freeradius/3.0/clients.conf` добавьте:
     ```plaintext
     client mikrotik {
       ipaddr = 192.168.1.1
       secret = mysharedsecret
     }
     ```
3. **Добавьте пользователей** (локальная база или интеграция с AD).
4. **Перезапустите службу**:
   ```bash
   sudo systemctl restart freeradius
   ```

## 2. Настройка MikroTik для работы с RADIUS
1. **Добавьте RADIUS-сервер**:
   ```bash
   /radius add service=wireless address=192.168.1.100 secret=mysharedsecret authentication-port=1812 accounting-port=1813
   ```
2. **Привяжите RADIUS-аутентификацию к Wi-Fi-интерфейсу**:
   ```bash
   /interface wireless security-profiles set default authentication-types=wpa2-eap eap-methods=peap-mschapv2 radius-mac-authentication=yes radius-mac-accounting=yes
   ```
3. **Включите RADIUS в настройках Wi-Fi**:
   ```bash
   /interface wireless set wlan1 security-profile=default mode=ap-bridge
   ```
4. **Проверьте соединение с RADIUS**:
   ```bash
   /radius monitor
   ```

## 3. Настройка Wi-Fi клиентов

### 3.1. Автоматическая настройка Wi-Fi через GPO (для доменных ПК)
1. **Откройте "Редактор групповых политик"** (`gpedit.msc`).
2. **Создайте новую политику** в разделе "Конфигурация компьютера" → "Параметры Windows" → "Сетевые параметры" → "Беспроводные сети (802.1X)".
3. **Настройте профиль Wi-Fi**:
   - Имя SSID.
   - Метод аутентификации **WPA2-Enterprise**.
   - Включите PEAP/MSCHAPv2.
4. **Примените политику и обновите её на клиенте**:
   ```powershell
   gpupdate /force
   ```

### 3.2. Ручная настройка Wi-Fi на Windows/macOS
1. **Добавьте новое соединение Wi-Fi**.
2. **Выберите метод аутентификации** → WPA2-Enterprise.
3. **Укажите логин/пароль доменного пользователя**.
4. **При необходимости загрузите сертификат RADIUS**:
   ```powershell
   certmgr.msc
   ```
5. **Примените настройки и подключитесь**.

## 4. Отладка и диагностика
1. **Проверка логов RADIUS на Windows**:
   ```powershell
   eventvwr.msc
   ```
2. **Проверка логов FreeRADIUS на Linux**:
   ```bash
   journalctl -u freeradius
   ```
3. **Тестирование аутентификации вручную**:
   ```bash
   radtest user password 127.0.0.1 1812 mysharedsecret
   ```
4. **Диагностика на MikroTik**:
   ```bash
   /log print where message~"radius"
   ```

## 5. Дополнительные возможности
- **Мониторинг подключений**: `/ip accounting print`
- **Принудительное отключение клиента**:
   ```bash
   /interface wireless registration-table remove [find where mac-address=XX:XX:XX:XX:XX:XX]
   ```
- **Ограничение скорости через RADIUS**:
   - В настройках RADIUS используйте атрибут `Mikrotik-Rate-Limit`.

## Заключение
Следуя этим шагам, вы сможете настроить безопасную Wi-Fi сеть с RADIUS-аутентификацией. Если возникают ошибки, проверяйте логи и диагностику!

