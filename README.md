# AreyaWeb: System Administration Project on Debian

**Authors**: Alberto García Martín, Mireya Fernández Gallego  
**Date**: May 20, 2024  
**Course**: System Administration  
**Environment**: Debian 12 (VirtualBox)

---

## 📚 Project Overview

This project involves the installation, configuration, and administration of a Linux server (Debian-based) designed to provide essential services for an academic and corporate infrastructure. The server includes:

- Web server (Apache2)
- Educational platform (Moodle)
- Blogging platform (WordPress)
- Mail server (Postfix, Dovecot, Roundcube)
- User authentication and registration via CGI scripts
- MariaDB database
- Automated backups
- System monitoring tools

---

## ⚙️ System Architecture & Configuration

### 🔧 Base System Setup

- **Hostname**: `areya`
- **Disk Partitioning**: 
  - `sda1`: for user data and quotas
  - `/var`: for web services and scripts
- **Static IP**: 192.168.1.38
- **Secure SSH**: port changed to 2203, login timeout enabled, sftp active

### 🌐 Web Server

- **Apache2**: Installed with Suexec for executing scripts as `www-data`
- **PHP**: Installed to enable dynamic content rendering
- **OpenSSL**: Self-signed certificate for HTTPS support

### 🛠️ Additional Modules

- **CGI**: Enabled `.cgi` script execution and `userdir` module
- **CPAN**: Perl libraries installed to support server-side scripts
- **MariaDB**: Database `areyadb` for user registration and management

---

## 👤 User Management

- **Groups**: `student` (GID 2000), `teacher` (GID 2001)
- **Admin user**: `admin` — member of `sudoers`, used for executing server scripts
- **Skel Directory**: Includes terms and conditions, symbolic links to services and resources

---

## 📄 Implemented Scripts

Perl CGI scripts handle the full user lifecycle:

| Script                | Functionality                                                             |
|-----------------------|---------------------------------------------------------------------------|
| `registro.cgi`        | Registers new users, sends email verification link                        |
| `confirmar.cgi`       | Activates user account and assigns role-based redirection                 |
| `direcciones.cgi`     | Redirects confirmed users to their respective dashboards                  |
| `iniciarSesion.cgi`   | User login via PAM authentication                                          |
| `cerrarSesion.cgi`    | Logs the user out of the system                                            |
| `modificarContra.cgi` | Allows password change with old-password validation                       |
| `modificarPerfil.cgi` | Updates name, surname, and email in the database                          |

**CPAN Libraries used**: `CGI`, `CGI::Session`, `DBI`, `Email::Send::SMTP::Gmail`, `Linux::usermod`, `MIME::Base64`, among others.

---

## 📬 Mail Server

- **Postfix**: Configured for local domain with mailbox and message size limits
- **Dovecot**: IMAP and POP3 support enabled
- **Roundcube**: Webmail interface, customized SMTP settings for local use

---

## 🧠 Web Platforms

### Moodle
- Installed via official tarball
- Database and configuration files set up manually
- Accessible through custom Apache `VirtualHost`

### WordPress
- Downloaded and extracted from official source
- Database and user setup included
- Configured to allow user-generated blogs and posts

---

## 🕵️ Administrator Monitoring

- Apache `server-status` module enabled
- Access restricted to a trusted subnet
- Real-time performance and service status viewable via web interface

---

## 🔄 Automated Tasks (Crontab)

- **Daily database backup** at 3:00 AM to `/backup`
- **HTML service status report** generated every minute
- **Cleanup of unconfirmed users** to maintain database integrity

---

## 🧪 Functionality & Testing

- Custom scripts validate service states and port availability
- Email verification tested with SMTP over Gmail
- Moodle and WordPress platforms fully operational for admin and regular users

---


## 📚 References & Resources

- [DigitalOcean: SSH Connections](https://www.digitalocean.com/community/tutorials/how-to-use-ssh-to-connect-to-a-remote-server-es)
- [YouTube: Apache, Roundcube, Moodle Setup](https://www.youtube.com)
- [CPAN: Perl Modules](https://metacpan.org)

---

This project provided valuable hands-on experience in server administration, Linux-based service deployment, and secure multi-user web application management.
