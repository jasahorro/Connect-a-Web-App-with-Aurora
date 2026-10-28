# Connect a Web App with Aurora — Cheat Sheet

> 📚 **Part 2 of 2** in my Aurora projects · ← Previous: [Aurora Database with EC2](https://github.com/jasahorro/Aurora-Database-with-EC2/blob/main/CHEATSHEET.md)

Lesson notes for reviewing later. The project write-up is in [README.md](README.md).

Part 1 covers the database setup (instance classes, clusters, security groups) in more depth.

## Contents

1. [What I built](#what-i-built)
2. [Key terms](#key-terms)
3. [EC2: the computer the web app runs on](#ec2-the-computer-the-web-app-runs-on)
4. [Creating the Aurora database](#creating-the-aurora-database)
5. [Clusters, writers, readers and endpoints](#clusters-writers-readers-and-endpoints)
6. [Connecting to EC2 (SSH and `chmod 400`)](#connecting-to-ec2-ssh-and-chmod-400)
7. [Installing the web server software](#installing-the-web-server-software)
8. [Connecting the web app to Aurora](#connecting-the-web-app-to-aurora)
9. [Testing with the MySQL CLI](#testing-with-the-mysql-cli)
10. [Build order and cleanup](#build-order-and-cleanup)
11. [Self-quiz](#self-quiz)

---

## What I built

- 🧱 **An Aurora MySQL database** created from scratch and connected to an EC2 instance.
- 💻 **An EC2 web server** (Amazon Linux 2023) running Apache and PHP.
- 🌐 **A PHP web app** (`SamplePage.php`) with a form that saves employee names and addresses into an `EMPLOYEES` table in Aurora and shows every stored record on the page.
- 🔗 **A check from the database side**: the same records viewed with the MySQL CLI.

```
  Browser ──HTTP──►  ┌──────────────────────────────┐  port 3306  ┌──────────────────────┐
                     │ EC2 web server               │ ──────────► │ Aurora MySQL cluster │
  You ──SSH (.pem)─► │ Apache + PHP + php-mysqli    │             │ DB: sample           │
                     │ /var/www/html/SamplePage.php │             │ table: EMPLOYEES     │
                     │ /var/www/inc/dbinfo.inc      │             │ writer (+ readers)   │
                     └──────────────────────────────┘             └──────────────────────┘
```

## Key terms

| Term | Plain meaning | Analogy |
|---|---|---|
| **EC2 instance** | A virtual computer you rent from AWS; here it runs the web app | Renting a computer instead of buying one |
| **Key pair** | The credentials for logging in to EC2; you keep the private `.pem` file | The keys to the house |
| **Public IPv4 DNS** | The address you use to reach the instance | The house's street address |
| **Relational database** | Data stored in tables of rows and columns, queried with SQL | A set of linked spreadsheets |
| **Engine type** | The core software that powers the database (here, Aurora MySQL) | The operating system, but for a database |
| **Engine version** | The exact version of that software, e.g. Aurora MySQL 3.05.2 (compatible with MySQL 8.0.32) | Which version of Windows you install |
| **Template** | Preset choices for the rest of the setup page; Dev/Test picks cheaper options | A "recommended settings" button |
| **Cluster** | A group of database copies working together so the data is always available | One team sharing one filing cabinet |
| **Endpoint** | The address your app uses to find the database | A website URL, but for a database |
| **Apache (`httpd`)** | The web server that sends pages to browsers | The waiter bringing dishes to the table |
| **PHP** | The language the web page is written in; it runs on the server | The kitchen that cooks each page to order |
| **MySQLi** | The PHP extension that lets PHP talk to MySQL/Aurora | The phone line between the kitchen and the pantry |
| **Nano** | A text editor that runs inside the terminal | Notepad, but in the terminal |

## EC2: the computer the web app runs on

- A web app has to run *on a computer*. Rather than buying one, you **rent a virtual one from AWS**: that's EC2. All the processing, data handling and user interactions happen there.
- **Key pair** = the keys to the instance. You need them to add, change or update anything on it.
- **Public IPv4 DNS + key pair** go together: one is the house's address, the other is the key. Neither is any use without the other.

**The launch Summary panel:**

| Field | What it means |
|---|---|
| 💻 Number of instances | How many machines you're running |
| 🎨 Software Image (AMI) | The operating system (Amazon Linux 2023) |
| ⚙️ Virtual server type | The CPU, memory and network package |
| 👮 Firewall | Controls inbound and outbound traffic; in EC2 this is a **security group** |
| 🪣 Storage | The instance's disk |

## Creating the Aurora database

- **Region matters:** create EC2 and Aurora in the **same Region**, or the networking gets very complicated.
- **Relational vs "normal":** there's no "normal" database. There are **relational** (SQL) and **non-relational** (NoSQL) databases.
- **Why Aurora?** It's for the big jobs: large scale, peak performance and uptime, thanks to **clusters**. Ordinary MySQL or Oracle is more generic and cheaper, and fine for smaller, less demanding workloads. (Fun fact: Coca-Cola uses Aurora for its global consumer data.)
- **Template:** **Dev/Test** picks lower-cost options for a learning project.
- **Instance class:** **Burstable** (`db.t*`) is cheap for a steady baseline with occasional spikes.

| Class | Best for | Analogy |
|---|---|---|
| **Burstable** — *used here* | A steady baseline with random bursts of traffic | A car with a turbo button for short bursts |
| **Serverless v2** | Pay for what you use; scales automatically with traffic | A taxi meter |
| **Memory optimized** | Heavy, memory-intensive workloads | A truck: expensive, but it carries a lot |

> The cluster takes **5+ minutes** to create. Time for a tea ☕

## Clusters, writers, readers and endpoints

- A **cluster** is a group of database copies that work together so your data is always available.
- **Writer** (primary) instance: handles all the writes (`INSERT`, `UPDATE`, `DELETE`).
- **Reader** instances (replicas): handle reads (`SELECT`) only. They share the read workload and act as backups.
- **Why separate them?** One writer keeps changes focused and controlled. Several readers spread out read traffic as the database grows. If the writer fails, a reader is **automatically promoted** to primary.
- **Endpoint:** the contact point where data flows in and out. Your app uses the **writer endpoint** because it needs to add data.

| Part | Job | Analogy |
|---|---|---|
| Writer | Changes the data | The one clerk allowed to write in the files |
| Readers | Look data up; step in if the writer fails | Clerks who can only read the files, and cover for the writer |
| Endpoint | How the app finds the database | The clerks' office address |

## Connecting to EC2 (SSH and `chmod 400`)

```bash
chmod 400 your-key.pem
ssh -i your-key.pem ec2-user@<Public IPv4 DNS>
```

**What `chmod 400` means:**

- `chmod` = **change mode**, i.e. change a file's permissions.
- Each digit is one group of users, in order: **owner, group, everyone else**.
- `4` = read-only, `0` = no permissions.
- So `400` means **only you can read the key, and nobody else can do anything with it**. SSH refuses to use a key that others can read.

## Installing the web server software

```bash
sudo dnf update -y
sudo dnf install -y httpd php php-mysqli mariadb105
```

| Part | Meaning |
|---|---|
| 🦸‍♀️ `sudo` | "Superuser do": run with admin (root) rights |
| 👷 `dnf` | The package manager for the instance's software |
| ☝️ `update` | Update all installed software |
| 👇 `install` | Install new software |
| 👍 `-y` | Automatically answer "yes" to prompts |
| 🕸️ `httpd` | Apache HTTP Server |
| 🧑‍💻 `php` | PHP |
| 💬 `php-mysqli` | The MySQL extension for PHP |
| 🏠 `mariadb105` | MariaDB, a MySQL-compatible package; here it gives you the `mysql` CLI for talking to Aurora |

## Connecting the web app to Aurora

1. **Make the folders yours.** Folders under `/var/www` belong to `root`, so saving files there gives **Permission denied**. Take ownership:

   ```bash
   sudo chown ec2-user:ec2-user <folder>
   ```

   - 👨‍🚒 `chown` = **change owner**.
   - 👨‍👩‍👧‍👦 `ec2-user:ec2-user` = set both the **user** and the **group** to `ec2-user` (you) instead of root.

2. **Create the connection details file** `/var/www/inc/dbinfo.inc` with nano. It holds the **Aurora endpoint, username, password and database name** (`sample`).
   - It lives in `inc/`, **outside** the public `html/` folder, so browsers can't load it and see the password.
   - Never commit a real `dbinfo.inc` to GitHub.

3. **Create the page** `/var/www/html/SamplePage.php`. It:
   - pulls in the details from `dbinfo.inc`
   - connects to Aurora with MySQLi
   - creates the `EMPLOYEES` table if it doesn't exist
   - shows a form to add a name and address
   - lists every stored record

4. Open `http://<Public IPv4 DNS>/SamplePage.php` in a browser.

**Nano basics:** type or paste into the file, then **Ctrl+O** (save) → **Enter**, and **Ctrl+X** (exit). Nano comes with Linux, but you'd have to install it yourself to use it in a Windows terminal.

**If you hit "Permission denied" again:**

1. Exit nano with **Ctrl+X**.
2. Run `ls -ld` to see who owns the folder.
3. Run `sudo chown ec2-user:ec2-user .` (the `.` means "this folder").
4. Paste and save the file again.

## Testing with the MySQL CLI

The **MySQL CLI** gives you direct access to all MySQL features from the terminal. It supports automation and is the usual way to manage a database from a remote server like EC2.

```bash
mysql -h <Aurora writer endpoint> -P 3306 -u <username> -p
```

```sql
USE sample;
SELECT * FROM EMPLOYEES;
```

If the rows you added through the web form show up here, the web app really is writing to Aurora.

## Build order and cleanup

1. Launch the EC2 instance (Amazon Linux 2023, new key pair). Note its **Public IPv4 DNS** and key pair name.
2. Create the Aurora MySQL cluster in the **same Region** (Dev/Test template, Burstable class), connected to the EC2 instance.
3. SSH in (`chmod 400` the key first).
4. Update the instance and install `httpd php php-mysqli mariadb105`.
5. Create `/var/www/inc/dbinfo.inc` with the endpoint and credentials.
6. Create `/var/www/html/SamplePage.php`.
7. Add records through the browser, then check them in the MySQL CLI.

**Cleanup (Aurora isn't in the Free Tier):** delete the DB instances and cluster → terminate the EC2 instance → delete the key pair → delete the security groups the setup created.

---

## Self-quiz

<details><summary>1. Why does the web app need an EC2 instance?</summary>

A web app has to run on a computer. EC2 is a virtual computer you rent from AWS, where all the processing, data handling and user interactions happen.
</details>

<details><summary>2. What two things do you need to connect to an EC2 instance?</summary>

The **Public IPv4 DNS** (the address) and the **key pair** `.pem` file (the key).
</details>

<details><summary>3. Why must EC2 and Aurora be in the same Region?</summary>

Otherwise the networking between them gets very complicated.
</details>

<details><summary>4. What's the difference between engine type and engine version?</summary>

The **type** is the database software itself (Aurora MySQL). The **version** is the exact release, e.g. Aurora MySQL 3.05.2, compatible with MySQL 8.0.32.
</details>

<details><summary>5. When would you choose Aurora over plain MySQL?</summary>

For large-scale workloads that need peak performance and uptime. Plain MySQL is cheaper and fine for smaller, less demanding ones.
</details>

<details><summary>6. What does the Dev/Test template do?</summary>

It presets the rest of the setup page with lower-cost options suited to development and testing.
</details>

<details><summary>7. Which SQL operations go to the writer, and which can a reader handle?</summary>

The writer handles `INSERT`, `UPDATE` and `DELETE` (and reads). Readers handle `SELECT` only.
</details>

<details><summary>8. What happens if the writer instance fails?</summary>

A reader is automatically promoted to become the new primary.
</details>

<details><summary>9. What is an endpoint?</summary>

The contact point your app uses to find the database and send it requests, like a URL for a website.
</details>

<details><summary>10. What does <code>chmod 400 key.pem</code> do?</summary>

Gives the owner read-only access and everyone else no access at all. SSH needs the key locked down like this.
</details>

<details><summary>11. Break down <code>sudo dnf install -y httpd php php-mysqli mariadb105</code>.</summary>

Run as admin (`sudo`), use the package manager (`dnf`) to `install`, auto-answer yes (`-y`): the Apache web server, PHP, PHP's MySQL extension, and MariaDB (which provides the `mysql` CLI).
</details>

<details><summary>12. You get "Permission denied" saving a file in <code>/var/www/html</code>. How do you fix it?</summary>

Exit nano (Ctrl+X), check the owner with `ls -ld`, run `sudo chown ec2-user:ec2-user .`, then save the file again.
</details>

<details><summary>13. What's in <code>dbinfo.inc</code>, and why is it in <code>/var/www/inc</code> instead of <code>/var/www/html</code>?</summary>

The Aurora endpoint, username, password and database name. Keeping it outside the public `html` folder means browsers can't load it and read the password.
</details>

<details><summary>14. What does <code>SamplePage.php</code> do?</summary>

It loads the connection details from `dbinfo.inc`, connects to Aurora, creates the `EMPLOYEES` table if needed, shows a form to add employees, and lists all stored records.
</details>

<details><summary>15. How do you prove the web app really saved data to Aurora?</summary>

Connect with the MySQL CLI, run `USE sample;` then `SELECT * FROM EMPLOYEES;`, and check that the rows you added in the browser are there.
</details>
