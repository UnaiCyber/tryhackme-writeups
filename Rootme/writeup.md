# TryHackMe – RootMe (Fácil)

## 📖 Introducción
RootMe es un laboratorio de TryHackMe catalogado con dificultad fácil. El acceso inicial se consiguió mediante una shell inversa tras realizar un bypass al filtro de subida de ficheros. Posteriormente, se identificó que Python estaba configurado con permisos SUID, lo que permitió escalar privilegios y obtener acceso como usuario root.
  

Este writeup documenta paso a paso el proceso seguido, incluyendo reconocimiento, explotación y escalada, con el fin de reforzar buenas prácticas de pentesting y servir como referencia técnica.  

> ⚠️ **Disclaimer ético**: Este writeup se ha realizado en un entorno seguro y educativo. No debe aplicarse en sistemas reales sin autorización expresa.

## 🔎 Reconocimiento

Se realizó un escaneo inicial con Nmap utilizando scripts por defecto y detección de versiones:
<img width="937" height="405" alt="image" src="https://github.com/user-attachments/assets/e18ca6bc-f191-4a21-8499-40c22aecffa7" />

## Nmap "IP" -sC -sV ##  

## 🧪 Cuestionario del lab

Durante el laboratorio RootMe se plantearon las siguientes preguntas técnicas, resueltas mediante reconocimiento activo y análisis de servicios:

- **¿Cuántos puertos están abiertos tras el escaneo inicial?**  
  `2`

- **¿Qué versión de Apache está en ejecución?**  
  `2.4.41`

- **¿Qué servicio está activo en el puerto 22?**  
  `SSH`




## 📂 Gobuster

Para enumerar directorios en el servidor web se utilizó la herramienta **Gobuster**:

gobuster dir -u http:/"IP" -w /usr/share/wordlists/dirb/common.txt




- **¿Cuál es el directorio oculto?**  
  `/panel/`


  ## 🌐 Web

Al acceder al directorio `/panel/`, se presenta una interfaz de subida de archivos. Este tipo de funcionalidad puede ser vulnerable si no se aplican correctamente los filtros de tipo de archivo y ejecución en el servidor.

---

El objetivo en este punto es ejecutar una **reverse shell** que nos permita obtener acceso remoto a la máquina. Para ello, se utiliza el script PHP de **Pentestmonkey**, ampliamente conocido en entornos de pentesting.

Antes de subirlo, se modifican los parámetros para que la shell se conecte a nuestra máquina atacante, donde estaremos escuchando con `netcat`.
<img width="463" height="212" alt="image" src="https://github.com/user-attachments/assets/d0bb15e2-955f-4505-bd24-bda5c72962a1" />

---

Al intentar subir el archivo con extensión `.php`, el servidor rechaza la petición mostrando un mensaje de error que indica que los archivos PHP no están permitidos. Esto revela la existencia de un filtro basado en la extensión del archivo, aunque no necesariamente en su contenido.
<img width="1918" height="784" alt="image" src="https://github.com/user-attachments/assets/1220092f-3f5a-4c17-a47d-08ccfbc81d97" />

---

Para evadir esta restricción, se renombra el archivo a `.phtml`, una extensión que también puede ser interpretada por el motor PHP en servidores mal configurados.

Al subir el archivo con esta nueva extensión, el servidor lo acepta correctamente ✅.
<img width="1919" height="800" alt="image" src="https://github.com/user-attachments/assets/c119fb7e-5165-4e79-a0b9-feb201811fc0" />

---

Este comportamiento nos permite continuar con el proceso de explotación, preparando el entorno para recibir la conexión inversa desde la máquina víctima.


## 🗂 Acceso a la reverse shell

Una vez subida la reverse shell con extensión `.phtml`, accedemos al directorio `/uploads/`, donde el servidor expone un listado de archivos públicos.
<img width="1071" height="522" alt="image" src="https://github.com/user-attachments/assets/8098526a-d9da-4f98-8ec1-db1a6d6e6017" />


En este listado aparece nuestro archivo `php-reverse-shell.phtml`, lo que indica que ha sido almacenado correctamente y es accesible desde el navegador.

---

Al hacer clic sobre el archivo, el servidor lo interpreta como código PHP y ejecuta la reverse shell. En paralelo, tenemos un listener activo en nuestra máquina atacante utilizando `netcat`:

<img width="1334" height="404" alt="image" src="https://github.com/user-attachments/assets/c7eeb1ca-949b-4276-a501-2eae0d0b099c" />



## 🔐 Escalada de privilegios

Una vez obtenida la shell como el usuario `www-data`, lo primero que hacemos es explorar el sistema en busca de información sensible.

---

Accedemos al directorio `/var/www` y encontramos el archivo `user.txt`, que contiene la primera flag del laboratorio. Esta flag confirma que hemos comprometido el entorno web con éxito.
<img width="645" height="202" alt="image" src="https://github.com/user-attachments/assets/5274e855-7f5f-4412-9f1d-4b356b8f6514" />

---

Con la flag en nuestro poder, iniciamos la fase de **escalada de privilegios** para obtener acceso como usuario root.

En este caso, optamos por buscar binarios con el bit **SUID** activado, ya que pueden permitir la ejecución de comandos con privilegios elevados si están mal configurados.

---



## 🏁 root.txt — Escalada con GTFOBins

Ejecutamos el siguiente comando para listar todos los archivos con permisos SUID en el sistema:
<img width="991" height="792" alt="image" src="https://github.com/user-attachments/assets/a5c10008-f071-4cf0-945e-25ff5982438b" />


Tras identificar un binario vulnerable con permisos SUID, consultamos el recurso [GTFOBins](https://gtfobins.github.io/) para verificar si puede ser utilizado para escalar privilegios.

---

GTFOBins es una base de datos que recopila técnicas de explotación para binarios comunes en sistemas Unix, especialmente cuando tienen permisos SUID, capacidades especiales o están mal configurados.

---
<img width="1090" height="363" alt="image" src="https://github.com/user-attachments/assets/6b651c33-7d46-4cbd-9b4e-54d3a09fec96" />

En este caso, el binario vulnerable permite ejecutar código arbitrario con privilegios elevados. Siguiendo la técnica documentada en GTFOBins, ejecutamos el binario con una instrucción que invoca una shell persistente:



