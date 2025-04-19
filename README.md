# Frappe CRM Local Development Setup on Kali Linux

This guide provides step-by-step instructions to set up Frappe CRM for local development on Kali Linux, based on a successful installation process.

## Prerequisites

Before starting, ensure you have the necessary packages installed. Open your terminal and run the following commands:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y git python3 python3-pip python3-venv redis-server mariadb-server libmariadb-dev nodejs npm pkg-config
sudo npm install -g yarn # Install yarn globally via npm
sudo apt install -y xvfb libfontconfig1 build-essential
```

**Note on `wkhtmltopdf`:** `wkhtmltopdf` is required for PDF generation in Frappe. It can sometimes be tricky to install directly from standard repositories on Kali. If the `sudo apt install -y wkhtmltopdf` command (part of the larger block above) fails, you may need to download a pre-compiled binary package for your Kali version from the official `wkhtmltopdf` website and install it manually using `sudo dpkg -i <downloaded_package.deb>` followed by `sudo apt --fix-broken install`.

## 1. Secure MariaDB

After installing `mariadb-server`, it's highly recommended to secure your installation. While the `mysql_secure_installation` script might not be directly in your PATH, you can often run it using its full path or the `mariadb-secure-installation` command.

First, ensure the MariaDB service is running:

```bash
sudo systemctl start mariadb
sudo systemctl status mariadb
```

Look for `Active: active (running)`. If it's not running, use `sudo systemctl start mariadb`.

Then, try running the secure installation script. Attempt these commands in order until one works:

```bash
sudo mysql_secure_installation
# If the above fails:
sudo mariadb_secure_installation
# If the above fails:
sudo /usr/bin/mysql_secure_installation
# If the above fails:
sudo /usr/bin/mariadb_secure_installation
```

Follow the prompts to secure MariaDB. **Set a strong root password** when asked. Remember this password.

**If the secure installation script is not found or you cannot run it:** You may need to manually secure MariaDB by logging in as the root user and running SQL commands. On a fresh install, you might be able to log in initially using `sudo mysql -u root` or `sudo mysql -u root -p` (and pressing Enter for no password). Once in the MariaDB prompt (`MariaDB [(none)]>`), you can set a root password and remove insecure defaults using `ALTER USER` and `DROP DATABASE test`, `DELETE FROM mysql.user` commands as discussed in our conversation.

## 2. Install Frappe Bench CLI using `pipx`

On Kali and other Debian-based systems with recent Python versions, you might encounter an "externally-managed-environment" error when using `pip3` directly. The recommended way to install command-line tools like `bench` globally is using `pipx`.

```bash
sudo apt install -y pipx
pipx ensurepath
```

Close and reopen your terminal, or run `source ~/.bashrc` (or your shell's equivalent config file) to update your PATH.

Now, install `frappe-bench` using `pipx`:

```bash
pipx install frappe-bench
```

Verify the installation:

```bash
bench --version
```

You should see the installed bench version.

## 3. Initialize Frappe Bench

Navigate to the directory where you want to create your bench (e.g., `~/projects/`) and run the `bench init` command:

```bash
cd ~/projects/
bench init frappe-bench
```

This creates the `frappe-bench` directory and sets up the basic environment.

Navigate into the newly created bench directory:

```bash
cd frappe-bench
```

## 4. Get the Frappe CRM Application

Download the Frappe CRM application code:

```bash
bench get-app crm [https://github.com/frappe/crm](https://github.com/frappe/crm)
```

## 5. Create a New Frappe Site

Create a new site for your CRM instance. Replace `datarhinocrm.localhost` with your desired site name (using `.localhost` is recommended for local development).

```bash
bench new-site datarhinocrm.localhost
```

You will be prompted for:
* **MySQL root password:** Enter the MariaDB root password you set in Step 1.
* **Administrator Password for `datarhinocrm.localhost`:** **Set a strong password for the 'Administrator' user of this site.** Remember this password to log in to the CRM web interface.
* **Install default apps:** Type `y` and press Enter.

## 6. Install Frappe CRM on the Site

Install the CRM application onto the site you just created:

```bash
bench --site datarhinocrm.localhost install-app crm
```

(Replace `datarhinocrm.localhost` if you used a different site name).

## 7. Install Process Manager (if needed)

If you encounter an `ERROR: No process manager found` when running `bench start`, you need to explicitly install `honcho` within the bench's virtual environment.

From the `~/projects/frappe-bench` directory, run:

```bash
source env/bin/activate
pip install honcho
deactivate
```

## 8. Start the Frappe Development Server

From the `~/projects/frappe-bench` directory, start the development server:

```bash
bench start
```

Keep this terminal window open while you are using the CRM.

## 9. Access Frappe CRM

Open your web browser and go to the address of your site, typically on port 8000:

```
[http://datarhinocrm.localhost:8000](http://datarhinocrm.localhost:8000)
```

(Replace `datarhinocrm.localhost` with your site name).

You should see the Frappe login page.

* **Username (Email):** `Administrator`
* **Password:** The administrator password you set in Step 5 when running `bench new-site`.

Log in with these credentials to access your Frappe CRM.

To stop the server, go back to the terminal running `bench start` and press `Ctrl + C`.
```# datarhino-crm
