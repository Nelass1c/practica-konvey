
# Лабораторный отчёт – День 12


**ФИО:** Гул Керим Аллалович
 
**Дата выполнения:** 14.06.2025

**Преподаватель:** Виталий Новиков 


---

## 1. Тема и цель работы

**Тема:** Дешифровка TLS-трафика

**Цели работы:**

- Изучить механизм дешифровки TLS-трафика с использованием сеансовых ключей.
- Освоить настройку переменных окружения (SSLKEYLOGFILE) для сохранения сеансовых ключей из браузера или других утилит.
- Получить практические навыки по импорту лог-файла с ключами в Wireshark для расшифровки захваченного трафика.
- Научиться анализировать расшифрованный HTTPS-трафик для поиска аутентификационных данных (токенов, логинов, паролей).
---

## 2. Теоретическая часть

*   **TLS (Transport Layer Security):** Криптографический протокол, обеспечивающий защищенную передачу данных между узлами в сети Интернет. Является преемником SSL. Основная задача — обеспечить конфиденциальность и целостность данных.
*   **TLS Handshake:** Процесс согласования параметров сессии между клиентом и сервером, включающий аутентификацию, обмен ключами и выбор шифронаборов.
*   **Сессионный ключ (Session Key):** Симметричный ключ, который генерируется для одной конкретной сессии связи и используется для шифрования всего трафика в рамках этой сессии.
*   **`SSLKEYLOGFILE`:** Специальная переменная окружения, которую поддерживают браузеры (Firefox, Chrome) и другие инструменты. Если эта переменная задана, приложение будет записывать сессионные ключи TLS в указанный файл в формате, понятном для Wireshark.
*   **Wireshark (TLS Decryption):** Wireshark имеет встроенную функцию дешифровки TLS-трафика. Для этого ему необходимо указать путь к файлу с сессионными ключами (pre-master secret log) в настройках протокола TLS.
*   **HTTP в TLS:** Протокол HTTP (прикладного уровня) инкапсулируется в защищенный TLS-протокол (транспортного уровня), образуя HTTPS. После дешифровки TLS-сессии в Wireshark можно увидеть содержимое HTTP-запросов и ответов в открытом виде.

---

### 3. Ход работы

Работа состояла из трех основных этапов: подготовка среды для перехвата сессионных ключей, захват и последующая дешифровка TLS-трафика для анализа HTTPS-содержимого.


#### 3.1. Используемые команды

Для хранения файла с сессионными ключами была создана специальная директория и пустой log-файл.

```bash
mkdir -p /home/ratka/secrets

# Создание пустого файла для ключей
touch /home/ratka/secrets/sslkeylog.log
```
Браузер Firefox был запущен из терминала с предварительно заданной переменной окружения `SSLKEYLOGFILE`. Это заставило браузер записывать все сессионные ключи TLS в указанный файл.

```bash
SSLKEYLOGFILE="/home/ratka/secrets/sslkeylog.log" firefox &
```

Параллельно с работой в браузере был запущен Wireshark для захвата всего сетевого трафика на интерфейсе `eth0`. Захваченный трафик был сохранен в pcap-файл.

#### 3.2. Скриншоты и захват трафика

После завершения захвата трафика в настройках Wireshark (Edit -> Preferences -> Protocols -> TLS) был указан путь к файлу с сессионными ключами: `/home/ratka/secrets/sslkeylog.log`.

![wireshark](https://raw.githubusercontent.com/Nelass1c/practica-konvey/main/day12/screenshots/sslkeys.jpg)

Это позволило Wireshark автоматически дешифровать TLS-сессии. В результате анализа дешифрованного трафика был найден HTTP POST-запрос на тестовый сайт, в теле которого в открытом виде передавались аутентификационные данные.

На скриншоте ниже виден результат дешифровки: Wireshark показывает содержимое пакета на прикладном уровне (Hypertext Transfer Protocol), где четко видны перехваченные логин (`username`) и пароль (`password`).

![Дешифрованный трафик с учетными данными](https://raw.githubusercontent.com/Nelass1c/practica-konvey/main/day12/screenshots/PAROL.jpg)

---

### 4. Результаты

*   Была успешно настроена среда для экспорта сессионных ключей TLS из браузера Firefox в `sslkeylog.log` с помощью переменной окружения `SSLKEYLOGFILE`.
*   Захвачен TLS-трафик, сгенерированный во время аутентификации на тестовом веб-сайте.
*   Путем импорта файла с ключами в Wireshark была успешно дешифрована TLS-сессия.
*   В результате анализа дешифрованного трафика был идентифицирован HTTP POST-запрос, в теле которого в открытом виде содержались переданные аутентификационные данные.
*   Захваченный и дешифрованный трафик сохранен в файл: [day12.pcapng](https://raw.githubusercontent.com/Nelass1c/practica-konvey/main/day12/day12.pcapng).

---

### 5. Выводы

*   **Закреплены навыки:** Освоен практический механизм дешифровки TLS-трафика в Wireshark при наличии сессионных ключей. Изучен метод перехвата этих ключей из браузера с помощью переменной `SSLKEYLOGFILE`. Наглядно продемонстрирована важность защиты сессионных ключей, так как их компрометация полностью нивелирует защиту, предоставляемую TLS.

*   **Что показалось сложным:** Настройка Wireshark для правильного импорта ключей и поиска нужного пакета в захваченном трафике требует внимательности и понимания структуры протоколов.

*   **Какой инструмент был наиболее полезен:** Ключевыми инструментами выступили **Wireshark** с его мощной функцией дешифровки TLS и переменная окружения **`SSLKEYLOGFILE`**, которая является фундаментальным элементом для всего процесса анализа зашифрованного трафика в лабораторных условиях.

---