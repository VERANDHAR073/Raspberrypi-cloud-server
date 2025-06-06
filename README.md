Raspberry Pi Cloud Server Setup (Nextcloud + Tailscale)

This guide helps you set up a private cloud on a Raspberry Pi using Nextcloud and secure remote access with Tailscale.

Step 1: Mount the External Hard Drive (Optional)

Run these commands to format and mount your hard drive:

lsblk
sudo fdisk -l
sudo mkdir /mnt/mydrive
sudo mount /dev/sda1 /mnt/mydrive
sudo chown -R pi:pi /mnt/mydrive
sudo nano /etc/fstab   # (optional) to auto-mount on boot
df -h

Step 2: Install and Configure Nextcloud

Update system packages:

sudo apt update && sudo apt upgrade -y

Install Apache web server:

sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2

Install MariaDB server and secure it:

sudo apt install mariadb-server -y
sudo mysql_secure_installation

Create Nextcloud database and user:

sudo mysql -u root -p

In the MySQL shell, run:

CREATE DATABASE nextcloud;
CREATE USER 'nextclouduser'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextclouduser'@'localhost';
FLUSH PRIVILEGES;
EXIT;

Install PHP and required extensions:

sudo apt install php libapache2-mod-php php-mysql php-xml php-mbstring php-zip php-curl php-gd php-intl php-bcmath -y

Download and configure Nextcloud:

cd /tmp
wget https://download.nextcloud.com/server/releases/latest.zip
sudo apt install unzip
unzip latest.zip
sudo mv nextcloud /var/www/html/

Set permissions for Nextcloud files:

sudo chown -R www-data:www-data /var/www/html/nextcloud
sudo chmod -R 755 /var/www/html/nextcloud

Step 3: Install and Configure Tailscale

Install Tailscale:

curl -fsSL https://tailscale.com/install.sh | sh

Start and enable Tailscale service:

sudo tailscale up
sudo systemctl enable tailscaled

Step 4: Host Your Cloud Online Using Tailscale Funnel

Run a local server (replace port 3000 with your web server port if different):

python3 -m http.server 3000

Start Tailscale funnel to expose the port:

sudo tailscale funnel 3000   # for directory access

or

sudo tailscale funnel 80     # for HTTP port

You will receive a URL like:

https://rkraspberrypi.yourtailnetname.ts.net/

Step 5: Add Trusted Domain in Nextcloud Config

Edit Nextcloud config file:

sudo nano /var/www/nextcloud/config/config.php

Add your Tailscale URL (and any IPs) in the 'trusted_domains' array, for example:

'trusted_domains' =>
  array (
    0 => '192.168.33.135',
    1 => '192.168.76.135',
    2 => '100.68.238.13',
    3 => '172.16.0.66',
    4 => '192.168.130.135',
    5 => 'rkraspberrypi.tail6f52bf.ts.net'
  ),

Save and exit.

Step 6: Restart Apache2

sudo systemctl restart apache2

Your Nextcloud server is now accessible through the Tailscale funnel URL.

---

End of Setup.
