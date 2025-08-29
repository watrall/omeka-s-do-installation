# Omeka S Digital Ocean Manual Installation Tutorial

This repository contains a step-by-step guide for installing [Omeka S](https://omeka.org/s/) on a [DigitalOcean Droplet](https://www.digitalocean.com/).  
It is written with **students, scholars, and cultural heritage professionals** in mind — people who are experts in their own fields but may not have much technical background.  

**Please note, this tutorial was designed for graduate students in the Cultural Heritage Informatics Initiative Graduate Fellowship.  When originally deployed, it was avaiable on the closed CHI Knowledgebase.  Its been moved to Github for public access**

The tutorial is designed to do two things at once:
1. Help you successfully install and run Omeka S.  
2. Teach you the basics of how servers work, how to connect to them, and how software like Omeka S fits together with web servers, databases, and the command line.  

If you follow it carefully, you will not only have a working Omeka S installation but also a clearer picture of how server-based web applications are managed.  

---

This guide walks you through a **manual install of Omeka S** on a fresh [DigitalOcean Droplet](https://www.digitalocean.com/).  
It is written for **students and scholars with little to no server experience**. Every step explains **what you are doing** and **why** you are doing it, so you not only get Omeka S running but also learn how servers and web applications work.

---

## Table of Contents
1. [What is Omeka S?](#what-is-omeka-s)
2. [What You’ll Need](#what-youll-need)
3. [Step 1: Create the Droplet](#step-1-create-the-droplet-your-small-cloud-server)
4. [Step 2: Connect to the Server with SSH](#step-2-connect-to-the-server-with-ssh)
5. [Step 3: Create a Safe Admin User](#step-3-create-a-safe-admin-user-recommended)
6. [Step 4: Turn on a Simple Firewall](#step-4-turn-on-a-simple-firewall)
7. [Step 5: Update the System](#step-5-update-the-system)
8. [Step 6: Install Apache (the Web Server)](#step-6-install-apache-the-web-server)
9. [Step 7: Install PHP (the-Language-Omeka-s-Runs-On)](#step-7-install-php-the-language-omeka-s-runs-on)
10. [Step 8: Install MySQL (the Database)](#step-8-install-mysql-the-database)
11. [Step 9: Download Omeka S](#step-9-download-omeka-s-into-your-web-folder)
12. [Step 10: Configure Omeka S Database Connection](#step-10-tell-omeka-s-how-to-reach-the-database)
13. [Step 11: Permissions for Files](#step-11-allow-apache-to-write-where-it-needs-to)
14. [Step 12: Configure Apache for Omeka S](#step-12-make-apache-serve-omeka-s-from-your-ip-or-domain)
15. [Step 13: Run the Web Installer](#step-13-run-the-omeka-s-web-installer)
16. [Step 14: Optional Domain and HTTPS](#step-14-optional-use-a-real-domain-and-https)
17. [Step 15: Verify and Explore Omeka S](#step-15-verify-the-system-add-modules-and-understand-omeka-s-features)
18. [Troubleshooting](#troubleshooting-cheatsheet)
19. [Quick Command Summary](#quick-command-summary-copy-paste-block)
20. [Official References](#reference-links-official)

---

## What is Omeka S?

[Omeka S](https://omeka.org/s/) is a web publishing platform designed for galleries, libraries, archives, and museums.  
Unlike Omeka Classic, **one installation of Omeka S can power multiple sites** that all share a common pool of items and metadata. It also:

- Uses **linked open data (JSON-LD)** for interoperability.
- Provides separate themes for each site.
- Can be extended with **modules** (plugins) such as [IIIF Server](https://omeka.org/s/modules/) or CSV import.
- Is ideal for institutions or projects running more than one collection-based website.

---

## What You’ll Need

- A [DigitalOcean account](https://www.digitalocean.com/).  
- A basic Droplet (Ubuntu 24.04 LTS, 1–2 GB RAM is fine).  
- Optional: a domain name (for easier access and HTTPS).  
- Patience: this guide is step-by-step but assumes no prior server background.

Omeka S official requirements: Linux, Apache with `AllowOverride All` and `mod_rewrite`, MySQL 5.7.9+ (or MariaDB 10.2.6+), PHP 7.4+ (works up through PHP 8.3). See: [Omeka S Manual](https://omeka.org/s/docs/user-manual/install/).

---

## Step 1: Create the Droplet (your small cloud server)

1. In the [DigitalOcean control panel](https://docs.digitalocean.com/products/droplets/how-to/create/), click **Create → Droplet**.
2. Choose **Ubuntu 24.04 LTS**.
3. Pick the $6/month plan (enough to start).
4. Choose a datacenter region close to you.
5. Select **Password** login for simplicity.
6. Name it `omeka-server` and click **Create Droplet**.

**Why.** A Droplet is just a virtual computer. It will run Omeka S and serve your sites to the world.

---

## Step 2: Connect to the Server with SSH

On your computer, you need a way to send commands to the server. This is done through something called the **terminal**:

- **Mac:** Open the **Terminal** application (search for "Terminal" in Spotlight).  
- **Linux:** Use your built-in Terminal application.  
- **Windows:** Open **PowerShell** (search for it in the Start menu).  

Now connect to your server using **SSH (Secure Shell)**. Replace `YOUR_SERVER_IP` with the IP address of your Droplet:

```bash
ssh root@YOUR_SERVER_IP
```

Enter the password you chose when creating the Droplet.

**Why.** The terminal is a text-based way to talk to computers. SSH is the safe, standard way to control a remote server.

---

## Step 3: Create a Safe Admin User (recommended)

```bash
adduser sammy
usermod -aG sudo sammy
```

Then log back in as that user:

```bash
exit
ssh sammy@YOUR_SERVER_IP
```

**Why.** It’s safer to work as a regular user with “sudo” powers, rather than root.

---

## Step 4: Turn on a Simple Firewall

```bash
sudo ufw allow OpenSSH
sudo ufw allow "Apache Full"
sudo ufw enable
```

**Why.** A firewall blocks unwanted traffic while allowing web (Apache) and SSH access.

---

## Step 5: Update the System

```bash
sudo apt update
sudo apt upgrade -y
```

**Why.** Updates bring in the latest security patches and fixes.

---

## Step 6: Install Apache (the Web Server)

```bash
sudo apt install -y apache2
sudo systemctl enable --now apache2
```

Check in your browser: `http://YOUR_SERVER_IP` should show the Apache default page.

**Why.** Apache serves web pages to visitors.

---

## Step 7: Install PHP (the Language Omeka S Runs On)

```bash
sudo apt install -y php libapache2-mod-php php-mysql php-xml php-mbstring php-gd php-imagick php-zip php-cli
sudo a2enmod rewrite
sudo systemctl restart apache2
```

**Why.** Omeka S runs on PHP and needs specific extensions for databases, XML, and images.

---

## Step 8: Install MySQL (the Database)

```bash
sudo apt install -y mysql-server
sudo systemctl enable --now mysql
```

Enter MySQL:

```bash
sudo mysql
```

Create a database and user:

```sql
CREATE DATABASE omeka_s;
CREATE USER 'omekauser'@'localhost' IDENTIFIED BY 'STRONG_PASSWORD';
GRANT ALL PRIVILEGES ON omeka_s.* TO 'omekauser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

**Why.** Omeka S needs a database to store all your items and metadata.

---

## Step 9: Download Omeka S into Your Web Folder

```bash
cd /var/www
sudo mkdir -p /var/www/omeka-s
sudo chown $USER:$USER /var/www/omeka-s
cd /var/www/omeka-s

wget https://github.com/omeka/omeka-s/releases/latest/download/omeka-s.zip
unzip omeka-s.zip
shopt -s dotglob
mv omeka-s*/* .
rm -rf omeka-s* omeka-s.zip
```

**Why.** This gets the official Omeka S release and puts it in the folder Apache will serve.

---

## Step 10: Tell Omeka S How to Reach the Database

```bash
sudo nano /var/www/omeka-s/config/database.ini
```

Paste and edit:

```
user     = "omekauser"
password = "STRONG_PASSWORD"
dbname   = "omeka_s"
host     = "localhost"
```

**Why.** This is how Omeka S connects to your MySQL database.

---

## Step 11: Allow Apache to Write Where It Needs To

```bash
sudo chown -R www-data:www-data /var/www/omeka-s/files
sudo find /var/www/omeka-s/files -type d -exec chmod 775 {} \;
sudo find /var/www/omeka-s/files -type f -exec chmod 664 {} \;
```

**Why.** Only the `files/` directory needs to be writable (for uploads and thumbnails). Everything else stays read-only.

---

## Step 12: Make Apache Serve Omeka S from Your IP or Domain

```bash
sudo nano /etc/apache2/sites-available/omeka-s.conf
```

Paste:

```
<VirtualHost *:80>
    ServerName YOUR_DOMAIN_OR_IP
    DocumentRoot /var/www/omeka-s

    <Directory /var/www/omeka-s>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/omeka-s_error.log
    CustomLog ${APACHE_LOG_DIR}/omeka-s_access.log combined
</VirtualHost>
```

Enable the site:

```bash
sudo a2ensite omeka-s.conf
sudo a2dissite 000-default.conf
sudo systemctl reload apache2
```

**Why.** This tells Apache where to find your Omeka S site.

---

## Step 13: Run the Omeka S Web Installer

In your browser, go to:

```
http://YOUR_DOMAIN_OR_IP/admin
```

Fill in:
- Admin username and password  
- Email  
- Site title  

**Why.** This sets up the initial user and site.

---

## Step 14: Optional: Use a Real Domain and HTTPS

- [Point your domain to DigitalOcean DNS](https://docs.digitalocean.com/products/networking/dns/getting-started/dns-registrars/).  
- Install free HTTPS certificates with Let’s Encrypt:

```bash
sudo apt install -y certbot python3-certbot-apache
sudo certbot --apache -d yourdomain.org -d www.yourdomain.org
```

**Why.** HTTPS makes your site secure and professional.

---

## Step 15: Verify the System, Add Modules, and Understand Omeka S Features

- In the admin, check **Settings → System information**.  
- Add modules like [CSV Import](https://omeka.org/s/modules/).  
- Remember: Omeka S lets you run **multiple sites** from one installation.

---

## Troubleshooting Cheatsheet

- **Database driver error**: make sure `php-mysql` is installed.  
- **URLs not working**: check Apache rewrite is enabled (`a2enmod rewrite`).  
- **Thumbnails missing**: ensure `php-gd` or `php-imagick` is installed.  
- **PHP version**: Omeka S supports PHP 7.4–8.3, but not 8.4 yet.

---

## Quick Command Summary (Copy-Paste Block)

```bash
# Create user and firewall
adduser sammy && usermod -aG sudo sammy
ufw allow OpenSSH && ufw allow "Apache Full" && ufw enable

# Update and install
sudo apt update && sudo apt upgrade -y
sudo apt install -y apache2 mysql-server
sudo apt install -y php libapache2-mod-php php-mysql php-xml php-mbstring php-gd php-imagick php-zip php-cli
sudo a2enmod rewrite && sudo systemctl restart apache2

# MySQL database
sudo mysql -e "CREATE DATABASE omeka_s; CREATE USER 'omekauser'@'localhost' IDENTIFIED BY 'STRONG_PASSWORD'; GRANT ALL PRIVILEGES ON omeka_s.* TO 'omekauser'@'localhost'; FLUSH PRIVILEGES;"

# Download Omeka S
cd /var/www && sudo mkdir omeka-s && sudo chown $USER:$USER omeka-s && cd omeka-s
wget https://github.com/omeka/omeka-s/releases/latest/download/omeka-s.zip
unzip omeka-s.zip && shopt -s dotglob && mv omeka-s*/* . && rm -rf omeka-s* omeka-s.zip

# Configure DB
echo -e "user = \"omekauser\"\npassword = \"STRONG_PASSWORD\"\ndbname = \"omeka_s\"\nhost = \"localhost\"" | sudo tee /var/www/omeka-s/config/database.ini

# Permissions
sudo chown -R www-data:www-data /var/www/omeka-s/files
```

---

## Reference Links (Official)

- **Omeka S**
  - [Home](https://omeka.org/s/)  
  - [Download](https://omeka.org/s/download/)  
  - [User Manual](https://omeka.org/s/docs/user-manual/install/)  
  - [GitHub Repo](https://github.com/omeka/omeka-s)  
  - [Modules](https://omeka.org/s/modules/)

- **DigitalOcean**
  - [Create a Droplet](https://docs.digitalocean.com/products/droplets/how-to/create/)  
  - [Initial Ubuntu Setup](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu)  
  - [Apache on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-22-04)  
  - [MySQL on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-20-04)  
  - [Firewall with UFW](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu)  
  - [Let’s Encrypt SSL](https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu)  
  - [DNS and Domains](https://docs.digitalocean.com/products/networking/dns/getting-started/dns-registrars/)


