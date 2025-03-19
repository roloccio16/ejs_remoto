# Port Knocking

Port knocking consiste en abrir y cerrar un puerto, en este caso el de ssh, mediante una combinacion de llamadas a otros puertos en cierto orden, actuando como una "contraseÃ±a" y triggereando asi un commando

## Instalamos knockd

Primero tenemos que instalar knockd

```bash
sudo apt update
sudo apt install knockd
```

## Activamos knockd

```bash
sudo nano /etc/default/knockd
```
Dentro de este archivo solo tienes que modificar esa linea de codigo de 0 a 1
```bash
START_KNOCKD=0
```
```bash
START_KNOCKD=1
```

## Configuramos knockd

```bash
sudo nano /etc/knockd.conf
```

Aqui deberemos de modificar el comando default que nos viene porque no nos ha funcionado el iptables y esto es mas sencillo, aparte de que es recomendable que cambiemos la secuencia default tanto para cerrar como para abrir, ya que mientras lo probaba he visto en los logs una ip haciendo knocks que no era la mia.
```bash
[options]
        UseSyslog

[openSSH]
        sequence = 7001,8001,9001
        seq_timeout = 5
        command = ufw allow 22/tcp && ufw reload
        tcpflags = syn

[closeSSH]
        sequence = 9001,8001,7001
        seq_timeout = 5
        command = ufw deny 22/tcp && ufw reload
        tcpflags = syn
```

## Reiniciar knockd

```bash
sudo systemctl restart knockd
```
miramos que este active

```bash
sudo systemctl status knockd
```

## Testeamos la configuracion

Aseguraros de tener denegado previamente el puerto 22

```bash
ufw deny 22/tcp
# ufw enable si es necesario
ufw reload
```
Prueba esto logicamente desde otra maquina o host
```bash
knock <IP_DEL_SERVIDOR> 7001 8001 9001
```
Eso deberia de abrirlo y ya podrias conectarte por ssh

lo mismo pero al contrario para cerrarlo

```bash
knock <IP_DEL_SERVIDOR> 9001 8001 7001
```

En mi caso en el archivo de configuracion de knockd en options pone " UseSyslog" por lo que solo con hacer

```bash
tail -f /var/log/syslog | grep knockd
```
podras monitorear todo lo que hace y es bastante facil de entender

si no en vez de UseSyslog puedes cambiarlo por

```bash
logfile = /var/log/knockd.log
```
o cualquier archivo de log que tu quieras pero crealo con antelacion
