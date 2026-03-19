# Steps to Install MySQL: Windows

1. Go to: [https://dev.mysql.com/downloads/installer](https://dev.mysql.com/downloads/installer)
2. Download the MSI Installer:
    - Two options are available:
      - Web Installer (~2 MB)
      - Full Installer (larger size)
    - It is recommended to use the Full Installer to download all components at once.
3. Choose Setup Type: After running the installer, select the "Full" setup type. This ensures you get both the MySQL Server and the MySQL Workbench (the graphical interface)
4. Check Prerequisites: Click "Execute" to install the necessary requirements and components.
5. Type and Networking:
    - Keep the default TCP/IP connectivity.
    - Ensure the Port is set to 3306 (the standard port for SQL operations).
    - Make sure "Open Windows Firewall ports for network access" is checked so your applications can communicate with the database.
6. Authentication Method: Use the recommended "Strong Password Encryption" (newer method) and click Next.
7. Accounts and Roles: Set a Root Password. This is for the main administrator account.

    > Important: Note this password down as you will need it to access your databases later

8. Windows Service: Keep "Configure MySQL Server as a Windows Service" checked. You can choose to start the server manually if you don't want it to slow down your system startup.
9. Apply Configuration: Click Execute to apply all settings. Once finished, click Next and then Finish to complete the installation.
10. Verification: Open MySQL Workbench, connect using your root password, and run a simple query like `SELECT VERSION();` to confirm everything is working.

## Other Key Information

- MySQL Workbench vs. Shell:
  - Workbench: A GUI tool for managing databases visually.
  - MySQL Shell: A command-line interface that supports three modes: SQL, JavaScript, and Python.
- Networking Concepts: The video explains that TCP handles how data is broken into packets, while IP handles the addressing (where the data goes).
- X Protocol (Port 33060): This is a newer protocol introduced in MySQL 8 that supports both SQL and NoSQL (JSON) operations.

## Fun Fact

- MySQL was created in 1995. It was bought by Sun Microsystems in 2008, which was later acquired by Oracle in 2010.
- MariaDB was created by the original MySQL developers as a "fork" because they were concerned Oracle might make MySQL less open-source.
- Open Source Status: Even though Oracle owns it, the MySQL Community Edition remains completely free and open-source.