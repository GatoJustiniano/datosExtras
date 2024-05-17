# Desplegar Odoo con wsl
- [Desplegar Odoo 14 en Ubuntu 18.04 usando Windows 10](#desplegar-odoo-14-en-ubuntu-1804-usando-windows-10)
  - [1. Instalar WSL en Windows 10 (versión 2004)](#1-instalar-wsl-en-windows-10-versión-2004)
    - [1.1 Habilitar WSL](#11-habilitar-wsl)
    - [1.2 Habilitar el uso de Virtual Machine](#12-habilitar-el-uso-de-virtual-machine)
    - [1.3 Descargamos el kernel de Linux](#13-descargamos-el-kernel-de-linux)
    - [1.4 Establecemos WSL 2 como versión por defecto](#14-establecemos-wsl-2-como-versión-por-defecto)
    - [1.5 Instalamos Ubuntu 18.04](#15-instalamos-ubuntu-1804)
    - [1.6 Crear credenciales para Ubuntu](#16-crear-credenciales-para-ubuntu)
    - [1.7 Instalar Windows Terminal (opcional)](#17-instalar-windows-terminal-opcional)
  - [2. Instalar Docker y su extension](#2-instalar-docker-y-su-extension)
    - [2.1 Instalar Docker Desktop](#21-instalar-docker-desktop)
    - [2.1 Instalar Remote - WSL para VS Code](#21-instalar-remote---wsl-para-vs-code)
  - [3. Desplegar Odoo 14 en Docker](#3-desplegar-odoo-14-en-docker)
    - [3.1 Iniciar PostgreSQL](#31-iniciar-postgresql)
    - [3.2 Iniciar Odoo](#32-iniciar-odoo)
    - [3.3 Acceder a la interfaz web de Odoo](#33-acceder-a-la-interfaz-web-de-odoo)
    - [3.4 Crear containers usando docker-compose](#34-crear-containers-usando-docker-compose)

## 1. Instalar WSL en Windows 10 (versión 2004)

Para verificar la version de Windows 10 usamos tecla Windows + R y escribimos `winver`

### 1.1 Habilitar WSL 

Abrimos PowerShell como administrador y luego ejecutamos:

> `dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart`

### 1.2 Habilitar el uso de Virtual Machine

Abrimos PowerShell como administrador y luego ejecutamos:

> `dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart`

Reiniciamos el computador.

### 1.3 Descargamos el kernel de Linux

Link: [WSL2 Linux kernel](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)

Luego se ejecuta el .msi

### 1.4 Establecemos WSL 2 como versión por defecto

Abrimos PowerShell como administrador y luego ejecutamos:

> `wsl --set-default-version 2`

### 1.5 Instalamos Ubuntu 18.04

Link: [Ubuntu 18.04 LTS](https://www.microsoft.com/store/apps/9N9TNGVNDL3Q)

Desde la Microsoft Store le damos `Get` y empezara a instalarse automaticamente.

### 1.6 Crear credenciales para Ubuntu

Una vez instalado Ubuntu 18.04, iniciamos el programa y lo primero será crear un usuario y una contraseña para la distribución.

### 1.7 Instalar Windows Terminal (opcional)

Link: [Windows Terminal]([https://link](https://docs.microsoft.com/en-us/windows/terminal/get-started))

Opcionalmente podemos instalar la nueva terminal de Windows 10 para utilizar mas comodamente WSL


## 2. Instalar Docker y su extension

### 2.1 Instalar Docker Desktop

Link: [Docker Desktop](https://hub.docker.com/editions/community/docker-ce-desktop-windows/)

Descargamos e instalamos Docker Desktop para Windows

Por defecto la instalación viene con la casilla para funcionar con WSL 2 activada. Luego reiniciamos el computador y ya podremos usar docker en Windows 10.

Docker Desktop por defecto se inicia con el sistema, pero esto se puede deshabilitar desde los ajustes.

### 2.1 Instalar Remote - WSL para VS Code

Link: [Remote - WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl)

Permite utilizar Visual Studio Code para ejecutar comando en la distro WSL de manera remota. 

Para verificar la instalación de docker podemos ejecutar el siguiente comando en una terminal de VS Code:

> `docker run hello-world`

## 3. Desplegar Odoo 14 en Docker

Para ejecutar Odoo en Docker necesitaremos crear 2 containers, para esto debemos usar 2 imagenes:

1. Odoo image: Odoo tiene una imagen oficial en docker hub. [Ver en Docker Hub](https://hub.docker.com/_/odoo).
2. PostgreSQL image: Odoo utiliza PostgreSQL para almacenar y manipular los datos. [Ver en Docker Hub](https://hub.docker.com/_/postgres).

### 3.1 Iniciar PostgreSQL

Ejecutamos:

> `docker run -d -e POSTGRES_USER=odoo -e POSTGRES_PASSWORD=odoo -e POSTGRES_DB=postgres --name db postgres:10`

Información del container:
> User: odoo<br>Password: odoo<br>Nombre de la BD: postgres<br>Nombre del container: db<br>Tag imagen: 10<br>Puerto: 5432<br>Tamaño imagen: 200 MB

### 3.2 Iniciar Odoo

Ejecutamos:

> `docker run -p 8069:8069 --name odoo --link db:db -t odoo`

Información del container:
> Container de la BD: db<br>Nombre del container: odoo<br>Tag imagen: latest<br>Puerto: 8069<br>Tamaño imagen: 1.18 GB

### 3.3 Acceder a la interfaz web de Odoo

Si la instalación es correcta en http://localhost:8069/ debemos poder ver el inicio de sesión de Odoo.

Para detener el container:

> `docker stop odoo`

Para empezar el container ya existente:

> `docker start -a odoo`


### 3.4 Crear containers usando docker-compose

Podemos crear un archivo `docker-compose.yml` donde declaremos las imagenes y variables de entorno que usaremos (en este ejemplo se usa odoo 11)

```yml
version: '2'
services:
  web:
    image: odoo:11
    depends_on:
      - mydb
    ports:
      - "8069:8069"
    environment:
    - HOST=mydb
    - USER=odoo
    - PASSWORD=myodoo
  mydb:
    image: postgres:10
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=odoo
      - POSTGRES_PASSWORD=myodoo
```

Estando en la misma carpeta que el archivo `docker-compose.yml` ejecutamos el comando

> `docker-compose up`
