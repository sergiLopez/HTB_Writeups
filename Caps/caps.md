
<div align="center">
  <img src="https://github.com/user-attachments/assets/b96bf5ae-f49f-4ead-94c5-c8ea807cd52a" alt="Escaneo de puertos">
</div>

# Análisis y Explotación de la Máquina

## Escaneo de Puertos

Primero de todo, empezaremos escaneando los puertos de la máquina:

<div align="center">
  <img src="https://github.com/user-attachments/assets/67238b26-7936-4895-8982-6531b278fc11" alt="Escaneo de puertos">
</div>

Encontramos tres puertos abiertos: un FTP, un SSH y un HTTP.

## Análisis de Versiones

Vamos ahora a analizar las versiones de estos servicios:

<div align="center">
  <img src="https://github.com/user-attachments/assets/4b218112-6ae6-436a-a148-fc4abf49e22b" alt="Análisis de versiones">
</div>

Después de intentar loguear anónimamente al FTP sin éxito, nos centramos en la versión del FTP para comprobar si existe alguna vulnerabilidad asociada:

<div align="center">
  <img src="https://github.com/user-attachments/assets/645de744-d57e-47c6-a8d8-211c43cb5aba" alt="Búsqueda de vulnerabilidades">
</div>

Vemos que existe un posible ataque DDoS, pero en este caso no nos interesa.

## Análisis del Sitio Web en el Puerto 80

Vamos ahora a analizar el sitio web que hay en el puerto 80.

Si nos dirigimos al apartado **Security Dashboard**, nos llama la atención que en esa ruta hay un número `4`:

<div align="center">
  <img src="https://github.com/user-attachments/assets/48391367-842f-4ce4-af0e-82c17fc5d640" alt="Security Dashboard">
</div>

Este sitio web nos da la posibilidad de descargarnos un archivo `.pcap`.

Si cambiamos el número de la ruta por `1, 2, 3, 4` y analizamos los archivos `.pcap`, no encontramos nada interesante. Sin embargo, si usamos el número `0`, la cosa cambia. Al abrir la captura, vemos lo siguiente:

`tshark -r 0.pcap -T fields -e tcp.payload 2>/dev/null | xxd -ps -r`

<div align="center">
  <img src="https://github.com/user-attachments/assets/e27dafc5-179a-4c25-acbf-6a6e28622f09" alt="Extracción de datos del archivo PCAP">
</div>

Vemos que el usuario `nathan` ha iniciado sesión en el servicio FTP con la contraseña `Buck3tH4TF0RM3!`. Después de comprobar esas credenciales, vemos que son correctas para el servicio FTP.

## Acceso SSH

Si probamos esas mismas credenciales para el servicio SSH, logramos conectarnos:

<div align="center">
  <img src="https://github.com/user-attachments/assets/a5358aba-1fc7-4043-89a1-a325b7f9af87" alt="Acceso SSH">
</div>

## Escalada de Privilegios

Nuestra misión ahora será escalar privilegios. Probamos archivos SUID y permisos `sudo`, pero sin éxito. Sin embargo, al revisar las **capabilities**, encontramos algo relevante:

<div align="center">
  <img src="https://github.com/user-attachments/assets/c6d19241-e1e9-4119-9e18-702a87927e01" alt="Análisis de capabilities">
</div>

Vemos que el binario de Python tiene la capability `cap_setuid`, que se puede explotar de esta manera:

<div align="center">
  <img src="https://github.com/user-attachments/assets/a02d3582-9d00-4422-af2e-9e477b4bae62" alt="Explotación de cap_setuid">
</div>



