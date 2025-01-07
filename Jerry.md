# Enumeración

El primer paso consiste en utilizar la herramienta `nmap` para identificar los puertos abiertos y los servicios activos.

![image](https://github.com/user-attachments/assets/7d7b18f8-5268-477c-8c44-5fa103290db7)

Es importante escanear todos los puertos con la opción `-p` y usar `--open` para centrarse en los puertos abiertos más comunes y acelerar el análisis.

En este caso, se detecta un servidor de aplicaciones **Apache Tomcat** operando en el puerto 8080.

![image](https://github.com/user-attachments/assets/342c1a4d-1ed6-4723-a94e-215754320e1b)

En evaluaciones internas, suelo priorizar herramientas como **phpMyAdmin** o **Apache Tomcat** como posibles puntos iniciales de compromiso. Esto se debe a que, frecuentemente, mantienen credenciales predeterminadas y ofrecen funcionalidades atractivas para el atacante.

# Explotación

El siguiente paso es verificar si el portal de inicio de sesión es accesible y, de ser así, probar credenciales por defecto. Para este propósito, se puede utilizar un diccionario disponible en el repositorio de [SecList](https://github.com/danielmiessler/SecLists).

En el caso de Apache Tomcat, las credenciales por defecto son: `tomcat/s3cret`.

Tras probarlas, se logró el acceso debido a un descuido de configuración:

![image](https://github.com/user-attachments/assets/0dd44e02-1731-4161-b5a2-da5693d3f3b6)

Además, se observa que ya se habían subido archivos previamente durante el uso del servicio 😉.

Con acceso autenticado, se puede aprovechar la función de subida de archivos de Tomcat (en formato `.war`) para cargar archivos `.jsp`. Para crear una shell en dicho formato, se puede usar `msfvenom`:

![image](https://github.com/user-attachments/assets/ce7597f9-5a80-44c8-a277-de93604af97f)

Posteriormente, se sube el archivo al servidor:

![image](https://github.com/user-attachments/assets/82848fd6-0311-44d6-b536-de740a1474e0)

En paralelo, se configura un listener en la máquina atacante para recibir la conexión. Por ejemplo, con netcat:

`nc -lnvp 6667`

Para activar la shell, basta con realizar una solicitud `GET` al archivo cargado: `http://10.10.10.95:8080/shell`:

![image](https://github.com/user-attachments/assets/bcabda1a-c958-47b3-81ff-64bf360deb43)

Esto genera una conexión entrante en el puerto configurado para escuchar:

![image](https://github.com/user-attachments/assets/9c561bb8-1f5e-493d-a7f8-d2b989b953cd)

# Postexplotación

Con el acceso conseguido, se verifica el usuario activo en el sistema:

![image](https://github.com/user-attachments/assets/68221e86-efe3-4dbf-924c-6088ed7368bf)

En este caso, se tiene acceso directo al usuario con mayores privilegios: **system**.

Si no fuera así, sería necesario proceder con la escalada de privilegios.

Finalmente, se accede al escritorio del usuario administrador y se localiza la flag:

![image](https://github.com/user-attachments/assets/3e41aa86-4606-4a23-a08b-9db299611fab)
