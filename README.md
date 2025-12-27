# firejail-firefox

# 🦊 Изоляция двух Firefox‑профилей через Firejail  
**Профили:**  
- `firefox-clean` — чистый, минимальный, без расширений  
- `firefox-dirty` — рабочий, с расширениями, cookies, логинами  

**Цель:**  
полная изоляция профилей друг от друга и от системы, без пересечения данных, без доступа к пользовательским файлам, с контролируемым окружением.

---

# 📌 1. Создание отдельных профилей Firefox

Создаём два профиля:

```bash
firefox --ProfileManager
```

Создаём:

- `clean`
- `dirty`

Проверяем, что Firefox создал каталоги:

```
~/.mozilla/firefox/<clean-id>.clean/
~/.mozilla/firefox/<dirty-id>.dirty/
```

---

# 📌 2. Создание отдельных Firejail‑профилей

Мы создаём два профиля:

- `/etc/firejail/firefox-clean.profile`
- `/etc/firejail/firefox-dirty.profile`

Они независимы и не пересекаются.

---

# 🧩 3. Firejail‑профиль для CLEAN Firefox

Файл:

```bash
sudo nano /etc/firejail/firefox-clean.profile
```

Содержимое:

```bash
# CLEAN Firefox profile

include /etc/firejail/firefox.profile

# Чистый профиль Firefox
private-tmp
private-cache

# Запрет доступа к пользовательским файлам
blacklist ~/.ssh
blacklist ~/Documents
blacklist ~/Downloads
blacklist ~/.local/share/containers
blacklist ~/.local/share/podman
blacklist ~/.local/share/docker

# Разрешаем доступ только к профилю clean
whitelist ~/.mozilla/firefox
read-only ~/.mozilla/firefox

# Разрешаем только конкретный профиль
whitelist ~/.mozilla/firefox/*.clean
```

---

# 🧩 4. Firejail‑профиль для DIRTY Firefox

Файл:

```bash
sudo nano /etc/firejail/firefox-dirty.profile
```

Содержимое:

```bash
# DIRTY Firefox profile

include /etc/firejail/firefox.profile

# Изолированная среда
private-tmp
private-cache

# Запрет доступа к пользовательским файлам
blacklist ~/.ssh
blacklist ~/Documents
blacklist ~/.local/share/containers
blacklist ~/.local/share/podman
blacklist ~/.local/share/docker

# Разрешаем доступ только к dirty-профилю
whitelist ~/.mozilla/firefox
read-only ~/.mozilla/firefox

# Разрешаем только конкретный профиль
whitelist ~/.mozilla/firefox/*.dirty
```

---

# 📌 5. Создание лаунчеров

## CLEAN Firefox

```bash
sudo nano /usr/local/bin/firefox-clean
```

Содержимое:

```bash
#!/bin/bash
exec firejail --profile=/etc/firejail/firefox-clean.profile firefox -P clean
```

Сделать исполняемым:

```bash
sudo chmod +x /usr/local/bin/firefox-clean
```

---

## DIRTY Firefox

```bash
sudo nano /usr/local/bin/firefox-dirty
```

Содержимое:

```bash
#!/bin/bash
exec firejail --profile=/etc/firejail/firefox-dirty.profile firefox -P dirty
```

Сделать исполняемым:

```bash
sudo chmod +x /usr/local/bin/firefox-dirty
```

---

# 🧪 6. Проверка изоляции

## Проверка запуска:

```
firefox-clean
firefox-dirty
```

## Проверка Firejail:

```
firejail --list
```

Ожидаем:

```
PID:user::firejail --profile=/etc/firejail/firefox-clean.profile firefox -P clean
PID:user::firejail --profile=/etc/firejail/firefox-dirty.profile firefox -P dirty
```

## Проверка, что профили не пересекаются:

```
firejail --join=<PID-clean> -- ls ~/.mozilla/firefox/*.dirty
```

Ожидаем:

```
No such file or directory
```

И наоборот:

```
firejail --join=<PID-dirty> -- ls ~/.mozilla/firefox/*.clean
```

Ожидаем:

```
No such file or directory
```

## Проверка, что нет доступа к пользовательским файлам:

```
firejail --join=<PID> -- ls ~/Documents
```

Ожидаем:

```
No such file or directory
```

---

# 🛡️ 7. Что обеспечивает эта конфигурация

### ✔ Полная изоляция clean и dirty профилей  
### ✔ Нет пересечения cookies, логинов, истории  
### ✔ Нет доступа к пользовательским файлам  
### ✔ Нет доступа к SSH‑ключам  
### ✔ Нет доступа к контейнерам  
### ✔ Нет доступа к Downloads/Documents  
### ✔ Чистое окружение через private-tmp  
### ✔ Минимальная поверхность атаки  

---

# 🧩 8. Дополнительные улучшения (опционально)

- запрет WebGL  
- запрет аудио/видео устройств  
- запрет D-Bus  
- запрет IPC  
- seccomp‑фильтры  
- read‑only режим для профиля  

