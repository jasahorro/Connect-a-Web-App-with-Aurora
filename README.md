<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Connect a Web App with Aurora

**Project Link:** [View Project](https://nextwork.ai/projects/921d7b78-ffa4-5e41-a5b5-aed76c78efad)

**Author:** Maria Jasmin Ahorro  
**Email:** jasminahorro23@gmail.com

---

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/921d7b78-ffa4-5e41-a5b5-aed76c78efad_1709b26b)

## Introducing Today's Project!

### What is Amazon Aurora?

Amazon Aurora is a fully managed relational database service built for the cloud, combining the speed and reliability of high-end commercial databases with the simplicity and cost-effectiveness of open-source solutions. It is completely compatible with MySQL and PostgreSQL, meaning existing applications and tools can run on Aurora with little to no modification.

It is useful because it addresses the traditional performance bottlenecks, scaling limitations, and high costs associated with standard database engines. Aurora features a fault-tolerant, self-healing distributed storage system that automatically scales storage across multiple Availability Zones, scaling up to 128 terabytes per database instance without requiring manual intervention. Furthermore, its distributed architecture replicates data six ways across three Availability Zones, ensuring high availability, continuous automated backups, and minimal downtime.

In enterprise environments, Aurora significantly boosts application performance, delivering up to five times the throughput of standard MySQL and up to three times the throughput of standard PostgreSQL. It separates compute from storage, allowing read replicas to handle heavy query workloads independently from the primary writer instance. By providing enterprise-grade security, automated patching, high scalability, and predictable performance, Amazon Aurora enables organizations to focus on building robust applications rather than managing complex database infrastructure.

### How I used Amazon Aurora in this project

In today's project, I used Amazon Aurora to create a scalable and reliable relational database cluster, configure secure connections from an EC2 web server, and successfully store and retrieve user-submitted web application data.

### One thing I didn't expect in this project was...

One thing I didn't expect in this project was how seamlessly Aurora automatically provisions a multi-instance cluster—handling the separation of read and write endpoints behind the scenes—while still allowing direct connection from an EC2 web server using standard PHP and MySQL tools.

### This project took me...

This project took me 45 minutes to complete! 

## Creating a Web App

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/921d7b78-ffa4-5e41-a5b5-aed76c78efad_b7999168)

To connect to my EC2 instance, I navigated to my desktop terminal where my .pem key file was saved and ran the SSH command with the private key and my EC2 instance's public IPv4 address.

To help me create my web app, I first installed an Apache web server, PHP, the PHP MySQLi extension, and MariaDB to enable database connectivity and handle server requests.

## Connecting my Web App to Aurora

I set up my EC2 instance's connection details to my database by creating an inc directory under /var/www, creating a dbinfo.inc configuration file using nano, and pasting my Aurora database endpoint, database username, password, and database name (sample).

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/921d7b78-ffa4-5e41-a5b5-aed76c78efad_1709b25b)

## My Web App Upgrade

Next, I upgraded my web app by creating a SamplePage.php file in my EC2 instance's HTML directory, pasting the complete PHP script that connects to the Aurora MySQL database, automatically creates the EMPLOYEES table if it does not exist, provides an input form to add employee names and addresses, and displays all stored records directly on the web page.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/921d7b78-ffa4-5e41-a5b5-aed76c78efad_2709b25b)

## Testing my Web App

To make sure my web app was working correctly, I tested it in my browser by adding sample employee data through the input form, and then verified that the records were successfully stored and retrieved from the database by querying the EMPLOYEES table using the MySQL CLI.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/921d7b78-ffa4-5e41-a5b5-aed76c78efad_1409z22b)

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/921d7b78-ffa4-5e41-a5b5-aed76c78efad)*
