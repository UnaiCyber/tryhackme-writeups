# TryHackMe – Dav (Fácil)

## 📖 Introducción
Dav es una máquina de TryHackMe con dificultad fácil. El acceso inicial se logró explotando el servicio WebDAV, que permitía la subida de ficheros y facilitó obtener una shell inversa. Posteriormente, se aprovechó una configuración insegura de sudo para escalar privilegios y conseguir acceso como root.


> ⚠️ **Disclaimer ético**: Este writeup se ha realizado en un entorno seguro y educativo. No debe aplicarse en sistemas reales sin autorización expresa.

---

## 🔎 Reconocimiento
Se realizó un escaneo inicial con Nmap utilizando scripts por defecto y detección de versiones:
<img width="967" height="230" alt="ksnip_20251215-195256" src="https://github.com/user-attachments/assets/4bf4799a-eca8-4b32-8fa2-babeba4ff5e7" />

Se realizó un escaneo con Nmap que reveló el puerto 80/tcp abierto, corriendo Apache 2.4.18 en Ubuntu. Al acceder desde el navegador, se mostró la página por defecto de Apache, lo que indica que el servidor está activo pero sin contenido personalizado


---


## 📂 Gobuster
Para enumerar directorios en el servidor web se utilizó la herramienta **Gobuster**:

gobuster dir -u http:/"IP" -w /usr/share/wordlists/dirb/common.txt

<img width="1108" height="647" alt="image" src="https://github.com/user-attachments/assets/b91b9c7d-756e-4d05-8d94-7b88ec67c07b" />


Aquí se detecta algo interesante: el directorio /webdav aparece en el escaneo de Gobuster con estado 401 Unauthorized, lo que indica que el servicio WebDAV está habilitado pero requiere autenticación.



## 🌐 Web


<img width="1919" height="860" alt="ksnip_20251215-211908" src="https://github.com/user-attachments/assets/83f4e847-762e-4ca0-ae54-2b0220d23ea2" />

Al entrar en la web, vemos que es una web de Apache por defecto. Vamos a probar a poner /webdav.

---

<img width="1500" height="648" alt="image" src="https://github.com/user-attachments/assets/e67bc203-c66e-45df-b106-395c7108abd4" />

Al acceder al directorio /webdav desde el navegador, el servidor responde con un prompt de autenticación, lo que confirma que el servicio WebDAV está activo y protegido por credenciales.

Investigando en fuentes abiertas, se identifica que en entornos mal configurados o de laboratorio, WebDAV puede usar credenciales por defecto. En este caso, se probaron las siguientes:
Usuario: wampp
Contraseña: xampp
Estas credenciales suelen estar asociadas a instalaciones de Apache con WebDAV habilitado en entornos de pruebas como XAMPP o WAMP. Al autenticarse correctamente, se habilita la posibilidad de subir archivos al servidor, lo que permite preparar una shell inversa para obtener acceso inicial.

---



## 🔑 Cadaver


<img width="875" height="470" alt="image" src="https://github.com/user-attachments/assets/3e8152a1-e9b8-45c2-a0c7-fb815b4cbea1" />




Se utilizó la herramienta cadaver para conectarse al servicio WebDAV en http://10.64.135.72/webdav. Al introducir las credenciales por defecto (wampp:xampp), el acceso fue exitoso y se obtuvo el prompt: dav:/webdav/>

Esto confirma que el servidor permite autenticación con credenciales débiles y que el usuario tiene permisos para interactuar con el directorio WebDAV. Desde aquí es posible subir archivos, lo que abre la puerta a cargar una shell inversa para obtener acceso inicial al sistema.


---


## 🧨 Ejecución de la reverse shell

---


<img width="1917" height="738" alt="ksnip_20251215-195322" src="https://github.com/user-attachments/assets/f8689ab5-9b2b-48f3-bbf9-aa7762b30f3b" />


Una vez dentro del servicio WebDAV, se subió un fichero llamado php-reverse-shell.phtml, que contiene una shell inversa en PHP. Este tipo de archivo permite ejecutar comandos en el servidor remoto y redirigir la salida hacia nuestra máquina atacante.






Antes de activar la shell, se lanzó un listener con Netcat para esperar la conexión entrante:


---

<img width="924" height="421" alt="ksnip_20251215-195324" src="https://github.com/user-attachments/assets/25851d48-2e5f-49c1-a0e2-bdd0fc0b2ddd" />




nc -lvnp 1234
Esto deja el sistema escuchando en el puerto 1234, preparado para recibir la conexión cuando se acceda al archivo malicioso desde el navegador. Al hacerlo, se establece una sesión interactiva con el sistema remoto, marcando el acceso inicial.



<img width="1210" height="516" alt="image" src="https://github.com/user-attachments/assets/0f692f32-4c6a-4cb3-8786-d943d7090e69" />



Tras subir el archivo php-reverse-shell.phtml al directorio WebDAV, se accedió a él desde el navegador haciendo clic en su URL. Esto activó el código PHP, que ejecutó una conexión inversa hacia nuestra máquina atacante.

Como ya se había lanzado el listener con Netcat (nc -lvnp 1234), la conexión fue recibida correctamente, estableciendo una shell remota como el usuario www-data. Desde este punto, se puede interactuar con el sistema objetivo y continuar con la fase de escalada de privilegios.


<img width="1225" height="510" alt="image" src="https://github.com/user-attachments/assets/68539073-f4e5-4b7e-9c54-8b6f269f35d5" />


Se accedió al directorio /home y se identificaron dos usuarios: merlin y wampp. Al entrar en /home/merlin y visualizar el contenido del archivo user.txt, se obtuvo la flag de usuario

## 🔐 Escalada de privilegios


<img width="1239" height="357" alt="image" src="https://github.com/user-attachments/assets/816457b5-97d0-453f-99cd-8bbc7507ac2d" />





🛡️ Escalada de privilegios con sudo

Se ejecutó el comando sudo -l para listar los permisos disponibles para el usuario www-data. El resultado mostró que este usuario podía ejecutar /bin/cat con sudo sin necesidad de contraseña:


(ALL) NOPASSWD: /bin/cat
Aprovechando esta configuración, se utilizó sudo cat para leer directamente el contenido de la flag de root ubicada en /root/root.txt:


sudo cat /root/root.txt
Este acceso confirma la escalada de privilegios exitosa, completando el compromiso total de la máquina.
