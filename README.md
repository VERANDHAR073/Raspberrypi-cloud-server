# ----------------------------------------
# Raspberry Pi Cloud Server Setup Script
# Nextcloud + Tailscale + Optional HDD
# ----------------------------------------

# STEP 1: Mount External Hard Drive (Optional)
lsblk
sudo fdisk -l
sudo mkdir /mnt/mydrive
sudo mount /dev/sda1 /mnt/mydrive
sudo chown -R pi:pi /mnt/mydrive
sudo nano /etc/fstab     # Add line to auto-mount on boot
df -h

# STEP 2: Install Nextcloud

# Update and Upgrade
sudo apt update && sudo apt upgrade -y

# Install Apache Web Server
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2

# Install MariaDB (Database)
sudo apt install mariadb-server -y
sudo mysql_secure_installation

# Install PHP and Extensions
sudo apt install php libapache2-mod-php php-mysql php-xml php-mbstring php-zip php-curl php-gd php-intl php-bcmath -y

# Create Nextcloud Database and User
sudo mysql -u root -p

# Inside MariaDB prompt:
# -----------------------
# CREATE DATABASE nextcloud;
# CREATE USER 'nextclouduser'@'localhost' IDENTIFIED BY 'your_password';
# GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextclouduser'@'localhost';
# FLUSH PRIVILEGES;
# EXIT;

# Download and Extract Nextcloud
cd /tmp
wget https://download.nextcloud.com/server/releases/latest.zip
sudo apt install unzip
unzip latest.zip
sudo mv nextcloud /var/www/html/

# Set Proper Permissions
sudo chown -R www-data:www-data /var/www/html/nextcloud
sudo chmod -R 755 /var/www/html/nextcloud

# STEP 3: Install and Configure Tailscale
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
sudo systemctl enable tailscaled

# STEP 4: Hosting the Cloud Online

# Option A: Serve a static directory (e.g., if hosting files directly)
python3 -m http.server 3000

# OR Option B: Use Apache for HTTP (default Nextcloud setup)
sudo systemctl restart apache2

# Expose the port securely via Tailscale Funnel
sudo tailscale funnel 3000   # If using python http server
# OR
sudo tailscale funnel 80     # If using Apache Nextcloud

# You’ll get a public URL like:
# https://yourhostname.tailxxxx.ts.net

# STEP 5: Configure Trusted Domains in Nextcloud

sudo nano /var/www/html/nextcloud/config/config.php

# Edit the 'trusted_domains' array like this:
# --------------------------------------------
# 'trusted_domains' =>
# array (
#   0 => 'localhost',
#   1 => '192.168.X.X',
#   2 => 'yourdomain.tailXXXX.ts.net',
# ),

# Also update overwrite.cli.url:
# 'overwrite.cli.url' => 'https://yourdomain.tailXXXX.ts.net',

# Restart Apache to apply changes
sudo systemctl restart apache2
