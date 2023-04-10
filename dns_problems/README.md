<h1 align="center">Обеспечение работоспособности приложения при включенном SELinux</h1>

<h2 align="left">Способ с переключателями setsebool</h2>
<p>
развернуть приложенный стенд https://github.com/mbfx/otus-linux-adm/tree/master/selinux_dns_problems<br>

чищу лог аудита
echo > /var/log/audit/audit.log

<img src="./screenshots/2.png"/><img>
На клиентской машине попробуем внести изменения в зону:<br>

```bash
nsupdate -k /etc/named.zonetransfer.key
```

<img src="./screenshots/1.png"/><img>

Изменения внести не получилось.<br>

Смотрим логи SELinux, чтобы понять в чем может быть проблема.<br>

```bash
cat /var/log/audit/audit.log | audit2why
```

Ошибок на клиенте нет.<br>

<img src="./screenshots/3.png"/><img>

Не закрывая сессию на клиенте, подключаюсь к серверу ns01
проверяю лог там


```bash
cat /var/log/audit/audit.log | audit2why
```

<img src="./screenshots/4.png"/><img>

В логах мы видим, что ошибка в контексте безопасности.<br>
Вместо типа named_t используется тип etc_t.<br>

проверим данную проблему в каталоге /etc/named<br>

```bash
ls -Z /etc/named
```

<img src="./screenshots/5.png"/><img>

посмотреть каталоги с подходящими политиками
sudo semanage fcontext -l | grep named

<img src="./screenshots/6.png"/><img>

chcon -R -t named_zone_t /etc/named
ls -laZ /etc/named
<img src="./screenshots/7.png"/><img>

снова вносим изменения с клиента
nsupdate -k /etc/named.zonetransfer.key
> server 192.168.50.10
> zone ddns.lab
> update add www.ddns.lab. 60 A 192.168.50.15
> send
> quit

<img src="./screenshots/8.png"/><img>


[root@client ~]# dig www.ddns.lab

<img src="./screenshots/9.png"/><img>

перезагружаю хосты
команда из методички показывает те же самые настройки
[root@client ~]# dig @192.168.50.10 www.ddns.lab
<img src="./screenshots/10.png"/><img>

старая же команда показывает другие
[root@client ~]# dig www.ddns.lab
<img src="./screenshots/11.png"/><img>


[root@ns01 ~]# restorecon -v -R /etc/named
<img src="./screenshots/12.png"/><img>


Обеспечить работоспособность приложения при включенном selinux.
развернуть приложенный стенд https://github.com/mbfx/otus-linux-adm/tree/master/selinux_dns_problems;
выяснить причину неработоспособности механизма обновления зоны (см. README);
предложить решение (или решения) для данной проблемы;
выбрать одно из решений для реализации, предварительно обосновав выбор;
реализовать выбранное решение и продемонстрировать его работоспособность.
К сдаче:
README с анализом причины неработоспособности, возможными способами решения и обоснованием выбора одного из них;
исправленный стенд или демонстрация работоспособной системы скриншотами и описанием.