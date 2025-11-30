এতক্ষণ আমরা থিওরি দেখলাম। এবার হাতে-কলমে কাজ শুরু করার সময়। এই চ্যাপ্টারে আমরা দেখব কীভাবে Nautobot ইনস্টল করতে হয়, সেটআপ করতে হয়। পুরো চ্যাপ্টারটা প্র্যাক্টিক্যাল হবে। আপনি এই চ্যাপ্টার পড়ে শেষ করার পরে আপনার নিজের Nautobot ইনস্ট্যান্স রানিং অবস্থায় থাকবে।

### আপনার প্রথম Nautobot ইনস্ট্যান্স

Nautobot ইনস্টল করার আগে একটা জিনিস ঠিক করতে হবে - আপনি কোন পদ্ধতিতে ইনস্টল করবেন। দুটো মূল পথ আছে। প্রথমটা হলো সরাসরি অপারেটিং সিস্টেমে ইনস্টল করা, দ্বিতীয়টা হলো Docker ইউজ করা। দুটো পদ্ধতিরই সুবিধা-অসুবিধা আছে।

আমি আপনাকে সাজেশন দেব Docker দিয়ে শুরু করার। কেন? কারণ Docker অনেক সহজ, দ্রুত, আর ঝামেলামুক্ত। আপনি দশ মিনিটে Nautobot চালু করতে পারবেন। তাছাড়া Docker ইউজ করলে আপনার মূল সিস্টেম ক্লিন থাকে, কোনো ডিপেন্ডেন্সি ইস্যু হয় না। তবে যারা Docker-এ স্বাচ্ছন্দ্য বোধ করেন না, তাদের জন্য আমি ট্র্যাডিশনাল ইনস্টলেশন পদ্ধতিও দেখাব।

### সিস্টেম রিকয়ারমেন্ট - কী কী লাগবে

প্রথমে দেখা যাক আপনার কী কী দরকার হবে। ছোট নেটওয়ার্কের জন্য (পাঁচ হাজার পর্যন্ত কাস্টমার) একটা মিডিয়াম সাইজ সার্ভার যথেষ্ট। আমি রেকমেন্ড করব:

**মিনিমাম রিকয়ারমেন্ট:**
- CPU: ৪ কোর (Intel/AMD x86_64)
- RAM: ৮ GB
- Storage: ৫০ GB SSD
- OS: Ubuntu 22.04 LTS অথবা Ubuntu 24.04 LTS

**রেকমেন্ডেড (১০ হাজার+ কাস্টমারের জন্য):**
- CPU: ৮ কোর
- RAM: ১৬ GB
- Storage: ১০০ GB SSD
- OS: Ubuntu 22.04 LTS অথবা Ubuntu 24.04 LTS

কেন Ubuntu? কারণ Ubuntu-তে Nautobot-এর সাপোর্ট সবচেয়ে ভালো। ডকুমেন্টেশনও Ubuntu-কেন্দ্রিক। CentOS/RHEL-এও চলে, কিন্তু Ubuntu দিয়ে শুরু করা সহজ। আর LTS (Long Term Support) ভার্সন নিন যাতে পাঁচ বছর সাপোর্ট পাবেন।

একটা ব্যাপার মনে রাখবেন - Nautobot নিজে খুব বেশি রিসোর্স খায় না। কিন্তু এর সাথে PostgreSQL ডেটাবেস, Redis ক্যাশ, আর Celery ওয়ার্কার চলে। এই সব মিলিয়ে রিসোর্স লাগে। তাই একটু ভালো সার্ভার নিন।

## Docker দিয়ে ইনস্টলেশন - সবচেয়ে সহজ পথ

Docker দিয়ে ইনস্টলেশন সবচেয়ে সহজ এবং দ্রুত। আমি ধরে নিচ্ছি আপনার কাছে একটা ফ্রেশ Ubuntu 24.04 LTS সার্ভার আছে। চলুন শুরু করি।

### স্টেপ ১: সার্ভার প্রস্তুত করা

প্রথমে আপনার সার্ভারে SSH করে লগইন করুন। তারপর সিস্টেম আপডেট করুন:

```bash
sudo apt update && sudo apt upgrade -y
```

এটা কয়েক মিনিট সময় নেবে। আপডেট শেষ হলে প্রয়োজনীয় কিছু প্যাকেজ ইনস্টল করুন:

```bash
sudo apt install -y curl git vim
```

### স্টেপ ২: Docker ইনস্টল করা

এখন Docker ইনস্টল করতে হবে। Ubuntu 24.04-এ Docker ইনস্টল করা খুবই সহজ:

```bash
# Docker-এর অফিশিয়াল GPG key যুক্ত করুন
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Docker রিপোজিটরি যুক্ত করুন
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Docker ইনস্টল করুন
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

ইনস্টলেশন শেষ হলে চেক করুন ঠিকমতো হয়েছে কিনা:

```bash
sudo docker --version
sudo docker compose version
```

আপনার ইউজারকে docker গ্রুপে যুক্ত করুন যাতে sudo ছাড়াই docker কমান্ড চালাতে পারেন:

```bash
sudo usermod -aG docker $USER
```

এখন একবার লগআউট করে আবার লগইন করুন। তারপর টেস্ট করুন:

```bash
docker ps
```

যদি কোনো এরর না আসে, মানে Docker ঠিকমতো কাজ করছে।

### স্টেপ ৩: Nautobot ডাউনলোড করা

এবার Nautobot-এর অফিশিয়াল Docker Compose ফাইল ডাউনলোড করব। একটা ডিরেক্টরি বানান:

```bash
mkdir -p ~/nautobot
cd ~/nautobot
```

এখন Nautobot-এর Docker Compose ফাইল ডাউনলোড করুন। সবচেয়ে সহজ উপায় হলো তাদের example রিপোজিটরি ক্লোন করা:

```bash
git clone https://github.com/nautobot/nautobot-docker-compose.git
cd nautobot-docker-compose
```

এই রিপোজিটরিতে সব প্রয়োজনীয় ফাইল আছে - Docker Compose কনফিগ, এনভায়রনমেন্ট ভ্যারিয়েবল, ইত্যাদি।

### স্টেপ ৪: কনফিগারেশন সেট করা

`nautobot-docker-compose` ডিরেক্টরিতে একটা `.env` ফাইল আছে। এটা হলো কনফিগারেশন ফাইল। এটা এডিট করতে হবে:

```bash
cp .env.example .env
vim .env
```

এখানে কয়েকটা গুরুত্বপূর্ণ জিনিস সেট করতে হবে:

```bash
# Nautobot ভার্সন (সবসময় latest stable version ইউজ করুন)
NAUTOBOT_VERSION=2.3.0

# Superuser credentials (প্রথম লগইনের জন্য)
NAUTOBOT_CREATE_SUPERUSER=true
NAUTOBOT_SUPERUSER_USERNAME=admin
NAUTOBOT_SUPERUSER_EMAIL=admin@example.com
NAUTOBOT_SUPERUSER_PASSWORD=YourStrongPasswordHere123!

# Secret key (একটা র‍্যান্ডম স্ট্রিং, ৫০+ ক্যারেক্টার)
SECRET_KEY=generate-a-random-secret-key-here-minimum-50-characters

# Database পাসওয়ার্ড
POSTGRES_PASSWORD=AnotherStrongPassword456!

# Redis পাসওয়ার্ড  
REDIS_PASSWORD=YetAnotherPassword789!
```

**গুরুত্বপূর্ণ:** পাসওয়ার্ডগুলো স্ট্রং রাখুন। প্রোডাকশনে দুর্বল পাসওয়ার্ড ইউজ করা মানে সিকিউরিটি রিস্ক।

SECRET_KEY জেনারেট করতে এই কমান্ড চালাতে পারেন:

```bash
python3 -c "import secrets; print(secrets.token_urlsafe(50))"
```

### স্টেপ ৫: Nautobot চালু করা

এবার সবচেয়ে রোমাঞ্চকর অংশ - Nautobot চালু করা। একটা কমান্ডই যথেষ্ট:

```bash
docker compose up -d
```

এটা প্রথমবার চালালে কয়েক মিনিট সময় নেবে। কারণ সব Docker ইমেজ ডাউনলোড হবে। আপনি দেখবেন এরকম আউটপুট:

```bash
[+] Running 5/5
 ✔ Container nautobot-redis-1     Started
 ✔ Container nautobot-db-1        Started
 ✔ Container nautobot-nautobot-1  Started
 ✔ Container nautobot-celery-1    Started
 ✔ Container nautobot-celery-beat-1 Started
```

চেক করুন সব কন্টেইনার চলছে কিনা:

```bash
docker compose ps
```

সবগুলো কন্টেইনারের স্ট্যাটাস "Up" দেখাবে।

এখন ব্রাউজার খুলে যান `http://your-server-ip:8080`। যদি সব ঠিক থাকে, তাহলে Nautobot-এর লগইন পেজ দেখতে পাবেন।

লগইন করুন আপনার সেট করা ইউজারনেম আর পাসওয়ার্ড দিয়ে (যেটা `.env` ফাইলে দিয়েছিলেন)। লগইন করার পরে আপনি Nautobot-এর হোম পেজ দেখতে পাবেন। অভিনন্দন! আপনার প্রথম Nautobot ইনস্ট্যান্স চালু হয়ে গেছে।

### স্টেপ ৬: প্রোডাকশন-রেডি সেটআপ

Docker দিয়ে যেভাবে চালু করলাম, সেটা ডেভেলপমেন্ট বা টেস্টিং-এর জন্য ঠিক আছে। কিন্তু প্রোডাকশনে ইউজ করতে হলে আরও কিছু কাজ করতে হবে।

**HTTPS সেটআপ করা:** প্রোডাকশনে HTTP দিয়ে চালানো উচিত না। আপনার একটা ডোমেইন লাগবে (যেমন `nautobot.yourisp.com`) আর একটা SSL সার্টিফিকেট। সবচেয়ে সহজ উপায় হলো Nginx Reverse Proxy ইউজ করা Let's Encrypt সার্টিফিকেট দিয়ে।

প্রথমে Nginx ইনস্টল করুন:

```bash
sudo apt install -y nginx certbot python3-certbot-nginx
```

একটা Nginx কনফিগ তৈরি করুন:

```bash
sudo vim /etc/nginx/sites-available/nautobot
```

এই কনফিগ লিখুন:

```bash
server {
    listen 80;
    server_name nautobot.yourisp.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

কনফিগ এনেবল করুন:

```bash
sudo ln -s /etc/nginx/sites-available/nautobot /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

এখন Let's Encrypt দিয়ে SSL সার্টিফিকেট নিন:

```bash
sudo certbot --nginx -d nautobot.yourisp.com
```

এটা অটোমেটিক্যালি Nginx কনফিগ আপডেট করে HTTPS সেটআপ করে দেবে।

**ব্যাকআপ সেটআপ:** প্রোডাকশনে ব্যাকআপ অত্যন্ত জরুরি। Nautobot-এর ডেটা মূলত PostgreSQL ডেটাবেসে থাকে। একটা ব্যাকআপ স্ক্রিপ্ট বানান:

```bash
vim ~/nautobot-backup.sh
```

এই স্ক্রিপ্ট লিখুন:

```bash
#!/bin/bash
BACKUP_DIR="/backup/nautobot"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Database backup
docker exec nautobot-db-1 pg_dump -U nautobot nautobot > $BACKUP_DIR/nautobot_$DATE.sql

# Keep only last 7 days backups
find $BACKUP_DIR -name "nautobot_*.sql" -mtime +7 -delete

echo "Backup completed: $DATE"
```

এক্সিকিউটেবল করুন:

```bash
chmod +x ~/nautobot-backup.sh
```

cron দিয়ে প্রতিদিন রাতে ব্যাকআপ চালান:

```bash
crontab -e
```

এই লাইন যুক্ত করুন:

```bash
0 2 * * * /home/youruser/nautobot-backup.sh >> /var/log/nautobot-backup.log 2>&1
```

এতে প্রতিদিন রাত ২টায় ব্যাকআপ হবে।

**মনিটরিং সেটআপ:** প্রোডাকশনে আপনি জানতে চাইবেন Nautobot ঠিকমতো চলছে কিনা। একটা সিম্পল হেলথ চেক স্ক্রিপ্ট বানাতে পারেন:

```bash
vim ~/check-nautobot.sh
```

```bash
#!/bin/bash
URL="http://localhost:8080/health/"
STATUS=$(curl -s -o /dev/null -w "%{http_code}" $URL)

if [ $STATUS -eq 200 ]; then
    echo "Nautobot is running fine"
else
    echo "Nautobot is down! Status: $STATUS"
    # এখানে আপনি email বা SMS নোটিফিকেশন পাঠাতে পারেন
fi
```

## ট্র্যাডিশনাল ইনস্টলেশন (Ubuntu 24.04)

যারা Docker ইউজ করতে চান না, তাদের জন্য ট্র্যাডিশনাল ইনস্টলেশন পদ্ধতি। এটা একটু বেশি সময় নেয়, কিন্তু আপনার সিস্টেমের উপর বেশি কন্ট্রোল পাবেন।

### স্টেপ ১: ডিপেন্ডেন্সি ইনস্টল

```bash
sudo apt update
sudo apt install -y python3.12 python3.12-venv python3-pip postgresql postgresql-contrib redis-server git libpq-dev
```

### স্টেপ ২: PostgreSQL সেটআপ

```bash
sudo -u postgres psql
```

PostgreSQL shell-এ:

```sql
CREATE DATABASE nautobot;
CREATE USER nautobot WITH PASSWORD 'YourStrongPassword';
ALTER ROLE nautobot SET client_encoding TO 'utf8';
ALTER ROLE nautobot SET default_transaction_isolation TO 'read committed';
ALTER ROLE nautobot SET timezone TO 'Asia/Dhaka';
GRANT ALL PRIVILEGES ON DATABASE nautobot TO nautobot;
\q
```

### স্টেপ ৩: Nautobot User তৈরি

```bash
sudo useradd -r -d /opt/nautobot -s /bin/bash nautobot
sudo mkdir -p /opt/nautobot
sudo chown nautobot:nautobot /opt/nautobot
```

### স্টেপ ৪: Virtual Environment তৈরি

```bash
sudo -u nautobot bash
cd /opt/nautobot
python3.12 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install nautobot
```

### স্টেপ ৫: কনফিগারেশন

```bash
nautobot-server init
```

এটা `/opt/nautobot/nautobot_config.py` ফাইল তৈরি করবে। এডিট করুন:

```bash
vim /opt/nautobot/nautobot_config.py
```

গুরুত্বপূর্ণ সেটিংস:

```python
ALLOWED_HOSTS = ['nautobot.yourisp.com', 'localhost']

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'nautobot',
        'USER': 'nautobot',
        'PASSWORD': 'YourStrongPassword',
        'HOST': 'localhost',
        'PORT': '',
        'CONN_MAX_AGE': 300,
    }
}

REDIS = {
    'default': {
        'HOST': 'localhost',
        'PORT': 6379,
        'PASSWORD': '',
        'DATABASE': 0,
    },
}

SECRET_KEY = 'your-generated-secret-key-here'
```

### স্টেপ ৬: Database Migration

```bash
nautobot-server migrate
```

### স্টেপ ৭: Superuser তৈরি

```bash
nautobot-server createsuperuser
```

### স্টেপ ৮: Static Files সংগ্রহ

```bash
nautobot-server collectstatic --no-input
```

### স্টেপ ৯: Systemd Service তৈরি

```bash
exit  # nautobot user থেকে বের হয়ে আসুন
sudo vim /etc/systemd/system/nautobot.service
```

```ini
[Unit]
Description=Nautobot WSGI Service
After=network.target postgresql.service redis.service

[Service]
Type=notify
User=nautobot
WorkingDirectory=/opt/nautobot
Environment="NAUTOBOT_ROOT=/opt/nautobot"
ExecStart=/opt/nautobot/venv/bin/nautobot-server start --pid-file /tmp/nautobot.pid
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Enable এবং start করুন:

```bash
sudo systemctl daemon-reload
sudo systemctl enable nautobot
sudo systemctl start nautobot
sudo systemctl status nautobot
```

এভাবে ট্র্যাডিশনাল ইনস্টলেশন শেষ। এখন Nginx দিয়ে reverse proxy সেটআপ করুন (আগে যেভাবে দেখিয়েছি)।

