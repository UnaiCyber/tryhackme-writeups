# TryHackMe – Dav (Fácil)

## 📖 Introducción
Dav es una máquina de TryHackMe con dificultad fácil. El acceso inicial se logró explotando el servicio WebDAV, que permitía la subida de ficheros y facilitó obtener una shell inversa. Posteriormente, se aprovechó una configuración insegura de sudo para escalar privilegios y conseguir acceso como root.


> ⚠️ **Disclaimer ético**: Este writeup se ha realizado en un entorno seguro y educativo. No debe aplicarse en sistemas reales sin autorización expresa.

## 🔎 Reconocimiento
Se realizó un escaneo inicial con Nmap utilizando scripts por defecto y detección de versiones:
<img width="967" height="230" alt="ksnip_20251215-195256" src="https://github.com/user-attachments/assets/4bf4799a-eca8-4b32-8fa2-babeba4ff5e7" />

Se realizó un escaneo con Nmap que reveló el puerto 80/tcp abierto, corriendo Apache 2.4.18 en Ubuntu. Al acceder desde el navegador, se mostró la página por defecto de Apache, lo que indica que el servidor está activo pero sin contenido personalizado


## 📂 Gobuster
Para enumerar directorios en el servidor web se utilizó la herramienta **Gobuster**:

gobuster dir -u http:/"IP" -w /usr/share/wordlists/dirb/common.txt

<img width="1108" height="647" alt="image" src="https://github.com/user-attachments/assets/b91b9c7d-756e-4d05-8d94-7b88ec67c07b" />


Aquí se detecta algo interesante: el directorio /webdav aparece en el escaneo de Gobuster con estado 401 Unauthorized, lo que indica que el servicio WebDAV está habilitado pero requiere autenticación.
