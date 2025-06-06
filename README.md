# Raspberry Pi Cloud Server Setup: Nextcloud + Tailscale

# --- Mount External Hard Drive (Optional) ---
lsblk
sudo fdisk -l
sudo mkdir /mnt/mydrive
sudo mount /dev/sda1 /mnt/mydrive
sudo chown -R pi:pi /mnt/mydrive
sudo nano /etc/fstab        # Add line to auto-mount
df -h

# --- Install and Configure Nextcloud ---

# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Apache
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2

# Install MariaDB
sudo apt install mariadb-server -y
sudo mysql_secure_installation

# Install PHP and required extensions
sudo apt install php libapache2-mod-php php-mysql php-xml php-mbstring php-zip php-curl php-gd php-intl php-bcmath -y

# Configure MariaDB for Nextcloud
sudo mysql -u root -p

# Inside MySQL shell:
# -------------------
# CREATE DATABASE nextcloud;
# CREATE USER 'nextclouduser'@'localhost' IDENTIFIED BY 'your_password';
# GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextclouduser'@'localhost';
# FLUSH PRIVILEGES;
# EXIT;

# Download and extract Nextcloud
cd /tmp
wget https://download.nextcloud.com/server/releases/latest.zip
sudo apt install unzip
unzip latest.zip
sudo mv nextcloud /var/www/html/

# Set permissions
sudo chown -R www-data:www-data /var/www/html/nextcloud
sudo chmod -R 755 /var/www/html/nextcloud

# --- Set Up Tailscale ---

# Install Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# Start and enable Tailscale
sudo tailscale up
sudo systemctl enable tailscaled

# --- Hosting the Cloud Online ---

# Option 1: Host with Python (static content or dev purposes)
python3 -m http.server 3000
sudo tailscale funnel 3000

# Option 2: Use Apache for Nextcloud and expose via Tailscale
sudo systemctl restart apache2
sudo tailscale funnel 80

# Example URL:
# https://your-hostname.tailYOURID.ts.net

# --- Configure Trusted Domains for Nextcloud ---

sudo nano /var/www/html/nextcloud/config/config.php

# Edit the 'trusted_domains' array:
# 'trusted_domains' => array (
#   0 => 'localhost',
#   1 => '192.168.x.x',
#   2 => 'your-tailscale-url.ts.net',
# ),

# Optionally update overwrite.cli.url:
# 'overwrite.cli.url' => 'https://your-tailscale-url.ts.net',

# Restart Apache
sudo systemctl restart apache2
