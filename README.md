# Сессионный проект «Безопасность GNU/Linux»

Настройка базовых средств защиты ОС GNU/Linux. Виртуальная машина с РЕД ОС 8.0.3 (Standard Server Minimal), без графического окружения, ядро 6.12. Все настройки сделаны из командной строки и сохраняются после перезагрузки. Итоговые файлы сняты после перезагрузки ВМ.

Номер зачётной книжки – 234.

## Как выполнен каждый пункт

| Пункт | Что сделано | Подтверждение |
|---|---|---|
| 1.1 Сеть | интерфейс enp0s3 получает адрес по DHCP: `nmcli connection modify enp0s3 ipv4.method auto connection.autoconnect yes` | history.out |
| 1.2 Имя хоста | `hostnamectl set-hostname mephi-2026.domain.local` | history.out, mephi-screenshot.png |
| 1.3 Проверка сети | `ping -c 4 8.8.8.8 > ping.out`, потерь нет | ping.out |
| 2.1 Обновление | `dnf -y update` | dnf.out |
| 2.2 Установка пакетов | `dnf -y install nginx libcap-ng-utils` | dnf.out |
| 2.3 Установка из rpm | `dnf download tcpdump` в /tmp, установка `rpm -ivh tcpdump-4.99.5-1.red80.x86_64.rpm` | history.out, tcpdump-4.99.5-1.red80.x86_64.rpm |
| 3.1 Второй диск | на /dev/sdb один раздел на весь диск (`parted`), `mkfs.ext4 -L MEPHI_WEB /dev/sdb1` | history.out |
| 3.2 Монтирование | точка /mephi-web, строка `LABEL=MEPHI_WEB /mephi-web ext4 defaults 0 2` в /etc/fstab, ручное `mount /mephi-web` | fstab, stat.out |
| 4.1 nginx | `systemctl enable --now nginx`, после перезагрузки сервис active | journalctl.out |
| 4.2 Журнал | `journalctl -u nginx -b > journalctl.out` | journalctl.out |
| 5.1 DAC | группа разработчиков mephi-dev (GID 5500), user1–user3 с UID 5501–5503, группа curators (GID 4444) с curator1 и curator2. Каталог /data/mephi-2026 владелец root:mephi-dev, права 2770 (setgid), ACL `g:curators:r-x` и ACL по умолчанию `g::rwx, g:mephi-dev:rwx, g:curators:r-x, o::---`. Новые файлы получают группу mephi-dev и права rw для команды, r для кураторов, ничего для остальных | stat.out, passwd, group, history.out |
| 5.2 tcpdump без set-UID | бит set-UID не используется, файлу выданы capabilities `setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump`, проверка `sudo -u user1 tcpdump -i enp0s3 -c 5 -n` | getcap.out, history.out |
| 5.3 SELinux | режим Enforcing (и в /etc/selinux/config). В nginx.conf `root /mephi-web;`, контекст каталога `semanage fcontext -a -t httpd_sys_content_t '/mephi-web(/.*)?'` и `restorecon -Rv /mephi-web`. До смены контекста nginx отдавал 403, после – страницу | getenforce.out, stat.out, curl.out |
| 6.1 Вход кураторов | в /etc/pam.d/login добавлен `account required pam_access.so`, в /etc/security/access.conf строка `-:(curators):LOCAL`. Вход curator1 с консоли отклоняется (`pam_access(login:account): access denied for user curator1 from tty1`), user1 входит | history.out |
| 6.2 Пароли | `PASS_MAX_DAYS 90` в /etc/login.defs и `chage -M 90` для всех пользователей, `minlen = 12` в pwquality.conf (проверка `pwscore`: пароль из 10 символов отклоняется) | shadow, pwquality.conf |
| 7 Проверка | /mephi-web/index.html с текстом `Hello from Student: 234`, `curl http://localhost/` | curl.out, mephi-screenshot.png |

## Файлы

| Файл | Содержимое |
|---|---|
| mephi-screenshot.png | консоль ВМ, `curl http://localhost/` выводит Hello from Student: 234 |
| history.out | история команд root |
| ping.out | `ping -c 4 8.8.8.8` |
| dnf.out | `dnf history` |
| stat.out | `stat /data/mephi-2026` и `stat /mephi-web` |
| journalctl.out | `journalctl -u nginx -b` |
| getcap.out | `getcap /usr/sbin/tcpdump` |
| getenforce.out | `getenforce` |
| curl.out | `curl http://localhost/` |
| fstab, passwd, shadow, group, pwquality.conf | системные файлы /etc/fstab, /etc/passwd, /etc/shadow, /etc/group, /etc/security/pwquality.conf |
| project_history.txt, mephi-nginx-screenshot.png | копии history.out и скриншота под именами из подсказки к заданию |
| tcpdump-4.99.5-1.red80.x86_64.rpm | пакет, скачанный `dnf download` и установленный через rpm |
