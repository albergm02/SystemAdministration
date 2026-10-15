<div align="center">

# 🖥️ AreyaWeb

**Infraestructura completa de servicios sobre Debian 12 — Administración de Sistemas**

[![Debian](https://img.shields.io/badge/Debian-12-A81D33?style=flat&logo=debian&logoColor=white)](https://www.debian.org/)
[![Apache](https://img.shields.io/badge/Apache-2.4-D22128?style=flat&logo=apache&logoColor=white)](https://httpd.apache.org/)
[![MariaDB](https://img.shields.io/badge/MariaDB-10.11-003545?style=flat&logo=mariadb&logoColor=white)](https://mariadb.org/)
[![Perl](https://img.shields.io/badge/Perl-CGI-39457E?style=flat&logo=perl&logoColor=white)](https://www.perl.org/)
[![Status](https://img.shields.io/badge/Status-Completado-1F7A8C?style=flat)]()

[Descripción](#-sobre-el-proyecto) · [Servicios](#-servicios-desplegados) · [Capturas](#-capturas) · [Arquitectura](#-arquitectura) · [Scripts CGI](#-gestión-de-usuarios-con-scripts-cgi) · [Autores](#-autores)

</div>

---

## 📖 Sobre el proyecto

**AreyaWeb** es un proyecto integral de **administración de sistemas** que despliega desde cero una infraestructura completa de servicios sobre **Debian 12** en una máquina virtual, simulando el entorno real de una empresa o institución académica.

El objetivo no era solo instalar servicios, sino **hacerlos convivir**: autenticación unificada de usuarios entre plataformas, verificación por correo electrónico, roles diferenciados, backups automatizados, monitorización en tiempo real y un sistema propio de gestión de usuarios escrito en **Perl**.

El proyecto cubre el ciclo completo: desde la instalación del sistema operativo hasta la puesta en producción de una plataforma educativa (Moodle) y una plataforma de blogs (WordPress), pasando por el correo electrónico y una base de datos central.

> 📌 **Proyecto académico** desarrollado para la asignatura *Administración de Sistemas* (curso 2023-2024).

---

## ⚙️ Servicios desplegados

| Servicio | Tecnología | Función |
| :--- | :--- | :--- |
| 🌐 **Servidor web** | Apache2 + Suexec | Alojamiento de servicios y ejecución de scripts como `www-data` |
| 🔐 **HTTPS** | OpenSSL | Certificado autofirmado para tráfico cifrado |
| 📚 **Plataforma educativa** | Moodle | Gestión de cursos, alumnos y recursos académicos |
| ✍️ **Plataforma de blogs** | WordPress | Sistema de blogs multiusuario |
| 📧 **Servidor de correo** | Postfix + Dovecot | Envío, recepción y almacenamiento de correo (SMTP/IMAP/POP3) |
| 📮 **Webmail** | Roundcube | Interfaz web para leer y gestionar el correo |
| 🗄️ **Base de datos** | MariaDB | Base de datos `areyadb` para usuarios y servicios |
| 👥 **Gestión de usuarios** | Scripts CGI en Perl | Registro, verificación por email, login, cambio de contraseña |
| 📊 **Monitorización** | Apache `server-status` | Panel de estado en tiempo real de servicios y rendimiento |
| 💾 **Backups** | `crontab` + `mysqldump` | Backups diarios de la base de datos |

## 🏗️ Arquitectura

### Configuración base del sistema

- **Hostname**: `areya`
- **IP estática**: `192.168.1.38`
- **Particionado**:
  - `sda1` → datos de usuario con cuotas
  - `/var` → servicios web y scripts
- **SSH securizado**:
  - Puerto cambiado a `2203` (evita escaneos automatizados)
  - Timeout de login configurado
  - SFTP activo

### Gestión de usuarios y grupos

- Grupos del sistema:
  - `student` — GID 2000
  - `teacher` — GID 2001
- Usuario administrador `admin` incluido en `sudoers`, encargado de ejecutar los scripts del servidor.
- Directorio `skel` personalizado con:
  - Términos y condiciones.
  - Enlaces simbólicos a servicios y recursos compartidos.

### Servidor web

- **Apache2** con **Suexec**: los scripts se ejecutan como `www-data`, evitando que un usuario comprometa el servidor.
- **PHP** instalado para dar soporte a Moodle y WordPress.
- **OpenSSL** con certificado autofirmado para HTTPS.
- Módulos habilitados: `cgi`, `userdir`, `ssl`.

---

## 👥 Gestión de usuarios con scripts CGI

El corazón del sistema es un conjunto de **scripts CGI en Perl** que gestionan el ciclo de vida completo del usuario: desde el registro hasta el cierre de sesión.

| Script | Función |
| :--- | :--- |
| `registro.cgi` | Registra nuevos usuarios y envía un correo de verificación. |
| `confirmar.cgi` | Activa la cuenta y redirige al usuario según su rol. |
| `direcciones.cgi` | Redirige a usuarios ya confirmados a su dashboard. |
| `iniciarSesion.cgi` | Login mediante autenticación **PAM**. |
| `cerrarSesion.cgi` | Cierra la sesión del usuario. |
| `modificarContra.cgi` | Cambio de contraseña validando la anterior. |
| `modificarPerfil.cgi` | Actualización de nombre, apellidos y correo en la base de datos. |

**Librerías CPAN utilizadas**: `CGI`, `CGI::Session`, `DBI`, `Email::Send::SMTP::Gmail`, `Linux::usermod`, `MIME::Base64`, entre otras.

### Flujo de registro

```
1. Usuario rellena formulario    →  registro.cgi
2. Se guarda en MariaDB          →  estado: pendiente
3. Se envía email de verificación
4. Usuario hace clic en el enlace →  confirmar.cgi
5. Cuenta activada + rol asignado
6. Redirección automática        →  direcciones.cgi
```

---

## 📬 Servidor de correo

- **Postfix**: configurado para dominio local con límites de tamaño de buzón y de mensajes.
- **Dovecot**: soporte **IMAP** y **POP3** para clientes de correo.
- **Roundcube**: webmail accesible desde el navegador, con SMTP ajustado para uso local.

Todo el flujo de correo se probó enviando emails reales a través de **Gmail SMTP** para la verificación de cuentas.

---

## 🧠 Plataformas web

### Moodle

- Instalado desde el tarball oficial.
- Base de datos y `config.php` configurados manualmente.
- Accesible a través de un **VirtualHost** dedicado en Apache.

### WordPress

- Descargado y extraído desde la fuente oficial.
- Base de datos y usuario propios.
- Configurado para permitir blogs y publicaciones generados por usuarios.

Ambas plataformas comparten la **autenticación unificada**: el usuario se registra una vez y accede a Moodle y WordPress con las mismas credenciales.

---

## 🕵️ Monitorización

- Módulo `server-status` de Apache habilitado.
- Acceso **restringido a una subred de confianza** (no es accesible desde cualquier IP).
- Vista en tiempo real del estado de los servicios, conexiones activas y carga del sistema.

---

## 🔄 Tareas automatizadas (Crontab)

| Tarea | Frecuencia | Descripción |
| :--- | :--- | :--- |
| **Backup de la base de datos** | Diario a las 3:00 AM | `mysqldump` de `areyadb` a `/backup`. |
| **Informe HTML del estado** | Cada minuto | Genera un HTML con el estado de los servicios. |
| **Limpieza de usuarios no confirmados** | Diario | Elimina cuentas pendientes para mantener la integridad. |

---

## 🛠️ Stack tecnológico

| Categoría | Tecnología |
| :--- | :--- |
| **Sistema operativo** | Debian 12 |
| **Servidor web** | Apache2, PHP, OpenSSL, Suexec |
| **Base de datos** | MariaDB |
| **Correo** | Postfix, Dovecot, Roundcube |
| **Plataformas** | Moodle, WordPress |
| **Scripting** | Perl (CGI, DBI, CGI::Session), Bash |
| **Autenticación** | PAM, validación por email |
| **Virtualización** | VirtualBox |
| **Red** | SSH (puerto 2203), SFTP |
| **Automatización** | Crontab |
| **Monitorización** | Apache server-status |

---

## 📂 Estructura del proyecto

```
SystemAdministration/
├── scripts/
│   ├── registro.cgi
│   ├── confirmar.cgi
│   ├── direcciones.cgi
│   ├── iniciarSesion.cgi
│   ├── cerrarSesion.cgi
│   ├── modificarContra.cgi
│   └── modificarPerfil.cgi
├── config/
│   ├── apache2/          # VirtualHosts y configuración
│   ├── postfix/          # main.cf y master.cf
│   ├── dovecot/          # Configuración de IMAP/POP3
│   └── crontab           # Tareas programadas
├── docs/
│   ├── memoria.pdf       # Memoria completa del proyecto
│   └── screenshots/
├── sql/
│   └── areyadb.sql       # Esquema de la base de datos
└── README.md
```

---

## 🚀 Cómo reproducirlo

1. **Instalar Debian 12** en una VM (VirtualBox, VMware o similar).
2. **Configurar la red**: IP estática `192.168.1.38`, hostname `areya`.
3. **Instalar los servicios base**:
   ```bash
   apt install apache2 php mariadb-server postfix dovecot-imapd dovecot-pop3d
   apt install libcgi-pm-perl libdbi-perl libcgi-session-perl
   ```
4. **Habilitar módulos de Apache**:
   ```bash
   a2enmod cgi userdir ssl suexec
   ```
5. **Importar el esquema de la base de datos**:
   ```bash
   mysql < sql/areyadb.sql
   ```
6. **Copiar los scripts CGI** a `/usr/lib/cgi-bin/` y darles permisos de ejecución.
7. **Configurar el crontab** con las tareas programadas.
8. **Reiniciar los servicios** y acceder a la IP desde el navegador.

> 📄 La memoria completa del proyecto está disponible en [`docs/memoria.pdf`](./docs/memoria.pdf).


## 📚 Referencias

- [DigitalOcean — Cómo usar SSH](https://www.digitalocean.com/community/tutorials/how-to-use-ssh-to-connect-to-a-remote-server-es)
- [CPAN — Repositorio de módulos Perl](https://metacpan.org)
- [Documentación oficial de Debian](https://www.debian.org/doc/)

---

## 👤 Autores

**Alberto García Martín**
- 🌐 Portfolio: [albertogm.dev](https://albertogm.dev)
- 💼 LinkedIn: [in/albertogarciamartin](https://www.linkedin.com/in/albertogarciamartin/)
- 🐙 GitHub: [@albergm02](https://github.com/albergm02)
- 📧 Email: alberto.g.m.0214@gmail.com

**Mireya Fernández Gallego**

---

<div align="center">

**⭐ Si este proyecto te resulta interesante, considera darle una estrella.**

</div>
