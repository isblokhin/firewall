#### Сценарии iptables
####   Задание
 - реализовать "knocking port": centralRouter может попасть на ssh inetRouter через knock-скрипт;
 - добавить inetRouter2, который виден с хоста (host-only network) или форвардится порт через localhost;
 - запустить nginx на centralServer и пробросить порт 80 на inetRouter2 порт 8080;
 - выход в Интернет в сети оставить через inetRouter. Дополнительно:
Дополнительно:
 - реализовать проход на 80й порт без маскарадинга.
