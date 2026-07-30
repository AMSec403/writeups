# Resolución de maquina DockerLabs: firsthacking

## información general.

* Plataforma: DockerLabs

* Objetivo: Explotar el servicio vsftpd 2.3.4 para ganar acceso root.

---

## Reconocimiento

Validamos conexión hacia la dirección IP de la victima y posteriormente realizamos un escaneo de puertos utilizando NMAP para identificar los servicios expuestos.

```bash
ping 10.88.0.2
nmap -sC -sV -oN firsthacking.txt 10.88.0.2
```

![reconocimiento.png](./images/reconocimiento.png)

Con esto logramos identificar que tenemos abierto el puerto 21, por el cual corre el servicio de vsftp 2.3.4

---

## Identificación de vulnerabilidades

Se realizo una búsqueda simple sobre el servicio vsftp 2.3.4 identificando la vulnerabilidad CVE-2011-2523, en donde el paquete distribuido entre el 30 de junio y el 3 de julio del 2011 incluía un backdoor que facilita abrir una shell sobre el puerto 6200 con privilegios elevados.

[Rapid7 Vulnerability Database](https://www.rapid7.com/db/modules/exploit/unix/ftp/vsftpd_234_backdoor/)

[vsftpd 2.3.4 - Backdoor Command Execution (Metasploit) - Unix remote Exploit](https://www.exploit-db.com/exploits/17491)

[NVD - CVE-2011-2523](https://nvd.nist.gov/vuln/detail/CVE-2011-2523)

![identificacion-01.png](./images/identificacion-01.png)

![identificacion-02.PNG](./images/identificacion-02.PNG)

![identificacion-03.PNG](./images/identificacion-03.PNG)

---

## Explotación

Tras identificar que existe una vulnerabilidad y un exploit, procedemos a buscarlo en Metasploit para verificar si viene cargado.

![explotacion-01.png](./images/explotacion-01.png)

Al ver que si se encuentra en la base de Metasploit procedemos a ejecutarlo para explotar la vulnerabilidad.

![explotacion-02.png](./images/explotacion-02.png)

```bash
msfconsole
search vsftpd
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 10.88.0.2
exploit
```

Al correr con éxito ganamos acceso a la shell, por lo que procedemos a validar el nivel de acceso, confirmando que tenemos acceso como root, que el usuario pertenece al grupo de root, validamos el directorio de trabajo actual, el cual corresponde al servicio comprometido y listamos todos todo el directorio raíz incluyendo ocultos en busca de algo mas interesante.

![explotacion-03.png](./images/explotacion-03.png)

---

## Mitigación

Tenemos una vulnerabilidad bastante conocida debido a que en 2011, un atacante comprometió el servidor donde se alojaba el paquete de instalación, suplantándolo por una versión maliciosa, la cual contenía un backdoor que facilitaba al atacante obtener acceso root sobre el puerto 6200.

Tras el hallazgo, el autor legitimo procedió a eliminar el paquete inmediatamente y publico las versiones corregidas, solicitando la actualización inmediata para todos los usuarios que hubieran descargado la versión entre el 30 de junio y 3 de julio de 2011.

Por lo que en este caso la corrección es simplemente realizar la actualización del paquete afectado.
