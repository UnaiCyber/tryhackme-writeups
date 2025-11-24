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
