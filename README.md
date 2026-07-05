# 🖥️ VPS Cheatsheet

> Полезные команды для управления и администрирования VPS сервера  
> 📌 **Репозиторий только для ознакомления** — не вносите изменения в чужие конфиги без понимания что делаете!

---

## 📦 Обновление системы

Обновить списки пакетов и установить обновления:

```bash
sudo apt update && sudo apt upgrade -y
```

Полная очистка системы (удаление ненужных пакетов и кеша):

```bash
sudo apt update && sudo apt upgrade -y && sudo apt autoremove && sudo apt autoclean
```

---

## 🔐 Безопасность

### Генерация надёжного пароля

```bash
openssl rand -base64 48
```

### Смена текущего пароля пользователя

```bash
passwd
```

---

## 🌍 Проверка GEO IP сервера

Скрипты для определения географического расположения IP сервера:

**Вариант 1:**
```bash
bash <(wget -qO- https://github.com/Davoyan/ipregion/raw/main/ipregion.sh)
```

**Вариант 2:**
```bash
bash <(wget -qO - https://github.com/vernette/ipregion/raw/master/ipregion.sh)
```

---

## 🌐 Отключение IPv6

### 🐧 На уровне ядра (sysctl)

Открой файл:

```bash
sudo nano /etc/sysctl.conf
```

Добавь в конец строки:

```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

Применить изменения и перезагрузить сервер:

```bash
sudo sysctl -p
sudo reboot
```

### 🔥 В файрволле UFW

```bash
sudo nano /etc/default/ufw
```

Найди `IPV6=yes`, замени на `IPV6=no`, затем:

```bash
sudo ufw reload
```

---

## 🛡️ Блокировка двухстороннего пинга (ICMP)

### 🔹 Беспантовый способ (sysctl)

Постоянное отключение ICMP Echo одной командой:

```bash
echo "net.ipv4.icmp_echo_ignore_all=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### 🔹 Жёсткий способ (UFW before.rules)

Открой конфиг:

```bash
sudo nano /etc/ufw/before.rules
```

Найди блоки `# ok icmp codes for INPUT` и `# ok icmp code for FORWARD`.  
Замени `ACCEPT` на `DROP` для `echo-request`. Добавь строку:

```
-A ufw-before-input -p icmp --icmp-type source-quench -j DROP
```

Перезагрузи UFW:

```bash
sudo ufw disable && sudo ufw enable
```

### 🔹 Мягкий способ (автоматический)

Всё одной командой:

```bash
# 1. Создаём бэкапы
sudo cp /etc/ufw/before.rules /etc/ufw/before.rules.bak.$(date +%s)
sudo cp /etc/ufw/before6.rules /etc/ufw/before6.rules.bak.$(date +%s)

# 2. Блокируем ping для IPv4
sudo sed -i '/echo-request/s/ACCEPT/DROP/' /etc/ufw/before.rules

# 3. Блокируем ping для IPv6
sudo sed -i '/echo-request/s/ACCEPT/DROP/' /etc/ufw/before6.rules

# 4. Перезагружаем UFW
sudo ufw reload

# 5. Проверяем результат
echo "=== IPv4 правила ==="
grep "echo-request" /etc/ufw/before.rules
echo "=== IPv6 правила ==="
grep "echo-request" /etc/ufw/before6.rules
```

---

## 🔨 Установка и настройка Fail2ban

Установка:

```bash
sudo apt update && sudo apt install fail2ban -y
```

Применить жёсткий конфиг (бан на 7 дней, 3 неудачные попытки за 3 минуты):

```bash
cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime = 7d
findtime = 3m
maxretry = 3
backend = systemd
banaction = iptables-multiport
allowipv6 = auto

[sshd]
enabled = true
mode = aggressive
port = 22,2222
EOF
```

Запуск и включение автозагрузки:

```bash
sudo systemctl enable fail2ban --now
```

Перезагрузка конфига:

```bash
sudo fail2ban-client reload
```

Проверка статуса и параметров:

```bash
fail2ban-client status sshd
fail2ban-client get sshd bantime
fail2ban-client get sshd maxretry
fail2ban-client get sshd findtime
```

---

## 🔍 Reality TLS Scanner

Поиск оптимального TLS-таргета для XTLS Reality:

```bash
# Скачать и запустить сканер
# Замени <server_ip> на IP твоего сервера
./RealiTLScanner-windows-64.exe -addr <server_ip>
```

> 🔗 [RealiTLScanner на GitHub](https://github.com/XTLS/RealiTLScanner)  
> После сканирования — **ручная проверка в браузере**, проверка пинга с сервера, выбор оптимального таргета.

---

## ⚠️ Важно

- Всегда делай **бэкапы** конфигов перед изменениями
- Понимай, что делает каждая команда, прежде чем её выполнять
- Данный репозиторий — шпаргалка, а не инструкция к бездумному копированию

---

<p align="center">
  🛡️ Безопасный админинг — осознанный админинг 🛡️
</p>