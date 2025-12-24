# Koha ILS Installation Guide

This repository provides a step-by-step guide and configuration snippets for installing **Koha Open Source Library LMS** on Debian-based systems (Ubuntu/Debian).

## 🚀 Prerequisites

* A Debian-based Linux distribution (Ubuntu 24.04+ or Debian 12 recommended).
* Root or sudo privileges.
* Stable internet connection.

## 🛠️ Installation Steps

### 1. Update and Prepare System

Ensure your system packages are up to date and install necessary transport tools.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y apt-transport-https ca-certificates curl sudo

```

### 2. Install and Secure MariaDB

Koha requires a database backend. MariaDB is the recommended choice for Debian-based systems.

```bash
sudo apt install -y mariadb-server
```

```bash
sudo mysql_secure_installation
```

Tip: During ``` mysql_secure_installation```, it is highly recommended to set a strong root password, remove anonymous users, and disallow root login remotely.

### What to expect during the script

The script will ask you 5-6 questions. For a Koha installation, follow these recommendations:

1. **Enter current password for root:** Since this is a new install, there is no password. Just press **Enter**.
2. **Switch to unix_socket authentication?** Usually, you can press **n** (No) here, as Koha often uses standard password authentication.
3. **Change the root password?** Press **Y** (Yes) and create a strong password. **Save this password safely**—you may need it if you ever need to manually fix database issues.
4. **Remove anonymous users?** Press **Y**. This prevents anyone from logging in without an account.
5. **Disallow root login remotely?** Press **Y**. This ensures that the 'root' user can only log in from the server itself, not from the internet.
6. **Remove test database?** Press **Y**. This removes a default database that is open to everyone.
7. **Reload privilege tables now?** Press **Y** to apply all changes immediately.

---

### ⚠️ Important Note for Koha Users

On modern systems (Ubuntu 22.04+ or Debian 12), MariaDB often uses `auth_socket` by default, meaning you log in with `sudo mariadb` without a password.

If your `koha-create` command fails later with "Access Denied," you may need to manually set the root password method by running:

```sql
sudo mariadb
-- Run this inside the MariaDB prompt:
ALTER USER 'root'@'localhost' IDENTIFIED VIA mysql_native_password USING 'YourPasswordHere';
FLUSH PRIVILEGES;
EXIT;

```

### 3. Configure Repositories

Add the Koha GPG key and the official software source.

##### Create keyring directory
```bash
sudo mkdir -p --mode=0755 /etc/apt/keyrings
```
##### Download Koha GPG key
```bash
sudo curl -fsSL https://debian.koha-community.org/koha/gpg.asc -o /etc/apt/keyrings/koha.asc
```
##### Add Koha LTS (24.11) source
```bash
sudo tee /etc/apt/sources.list.d/koha.sources <<EOF
Types: deb
URIs: https://debian.koha-community.org/koha/
Suites: 24.11
Components: main
Signed-By: /etc/apt/keyrings/koha.asc
EOF

```

> **Note:** You can check for other versions on the [Koha Wiki](https://wiki.koha-community.org/wiki/Koha_on_Debian).

### 4. Install Koha

```bash
sudo apt update
sudo apt install -y koha-common

```

### 5. Configuration

Configure the site instances and Apache modules.

* **Edit Site Config:**
```bash
sudo mousepad /etc/koha/koha-sites.conf

```


*Set `INTRAPORT` to **8080** and `OPACPORT` to **80**.*
* **Setup Apache:**
```bash
sudo a2enmod rewrite cgi
sudo systemctl restart apache2

```


* **Create Instance:**
```bash
sudo koha-create --create-db library

```


### 6. Network & Port Setup

If you are doing an IP-based installation, you must tell Apache to listen on port 8080.

```bash
sudo mousepad /etc/apache2/ports.conf

```

Add `Listen 8080` below the `Listen 80` line. Then, enable the virtual host:

```bash
sudo a2dissite 000-default
sudo a2enmod deflate
sudo a2ensite library
sudo systemctl restart apache2

```

## 🔐 Accessing the Web Interface

Once the services restart, access your Koha instance via your browser:

| Interface | URL |
| --- | --- |
| **OPAC (Public)** | `http://127.0.0.1` |
| **Staff Client (Admin)** | `http://127.0.0.1:8080` |

### Getting Your Credentials

To log in to the Staff Client for the first time:

* **Username:** `koha_library`
* **Password:** Retrieve it by running:
```bash
sudo koha-passwd library

```



## 🏁 Post-Installation

Follow the on-screen Web Installer wizard to:

1. Set up database tables.
2. Configure MARC frameworks.
3. Create a Library and a Super Librarian account.
