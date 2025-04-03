# Uso de ProxyChains con Túneles SSH para Anonimizar la IP

## Introducción
`ProxyChains` permite encadenar proxies para enrutar el tráfico de una aplicación a través de múltiples servidores y ocultar la IP real del usuario. En este documento, configuraremos `proxychains` para utilizar un túnel SSH como proxy.

Antes de comenzar, hemos creado varios VPS en Linode que utilizaremos para establecer los túneles SSH.

---

## Instalación de ProxyChains

### En Debian/Ubuntu
```bash
sudo apt update && sudo apt install proxychains
```

## Configuración de ProxyChains
El archivo de configuración de `proxychains` se encuentra en:
```bash
/etc/proxychains.conf  # Configuración global
~/.proxychains/proxychains.conf  # Configuración por usuario
```
Edita el archivo y ajusta lo siguiente:

1. Comenta la línea `dynamic_chain` y descomenta `strict_chain` para asegurar el uso de los proxies en orden.
2. Agrega tu servidor SSH como un proxy SOCKS5 al final del archivo:
```bash
socks5 127.0.0.1 5555
socks5 127.0.0.1 5556
socks5 127.0.0.1 5557

```

---

## Creación del Túnel SSH
Para establecer el túnel SSH, usa el siguiente comando, uno detras de otro:
```bash
ssh -D 5555 root@172.233.116.217
ssh -D 5556 root@172.233.116.42
ssh -D 5557 root@172.233.116.76
```

Explicación de opciones:
- `-D 5555`: Establece un proxy SOCKS en el puerto 5555.
---

## Uso de ProxyChains con Aplicaciones

O para realizar una solicitud con `curl`:
```bash
proxychains curl ifconfig.me
```

Si la IP mostrada es la del servidor remoto, la configuración es correcta.
 proxychains curl ifconfig.me
ProxyChains-3.1 (http://proxychains.sf.net)
|DNS-request| ifconfig.me 
|S-chain|-<>-127.0.0.1:5555-<>-127.0.0.1:5556-<>-127.0.0.1:5557-<><>-4.2.2.2:53-<><>-OK
|DNS-response| ifconfig.me is 34.160.111.145
|S-chain|-<>-127.0.0.1:5555-<>-127.0.0.1:5556-<>-127.0.0.1:5557-<><>-34.160.111.145:80-<><>-OK
172.233.116.76

---

## Conclusión
Con `proxychains` y un túnel SSH, podemos ocultar nuestra IP y mejorar la privacidad de nuestra conexión. Esta técnica es útil para evitar bloqueos geográficos, mejorar la seguridad en redes públicas y mantener el anonimato en línea.
