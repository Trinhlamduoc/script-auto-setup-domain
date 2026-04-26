## Script auto setup domain
#### Please read the full content. 
```
#!/bin/bash

DOMAIN="3d-model.dangtrinh.name.vn"
PORT="4000"
EMAIL="admin@$DOMAIN"

NGINX_CONF="/etc/nginx/sites-available/$DOMAIN"
NGINX_LINK="/etc/nginx/sites-enabled/$DOMAIN"

echo "=== Install Nginx + Certbot ==="
sudo apt update
sudo apt install -y nginx certbot python3-certbot-nginx

echo "=== Create Nginx config ==="

sudo tee $NGINX_CONF > /dev/null <<EOL
server {
    listen 80;
    server_name $DOMAIN;

    location / {
        proxy_pass http://127.0.0.1:$PORT;
        proxy_http_version 1.1;

        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host \$host;
        proxy_cache_bypass \$http_upgrade;
    }
}
EOL

echo "=== Enable site (safe) ==="

# Xóa symlink cũ nếu tồn tại
if [ -L "$NGINX_LINK" ]; then
    sudo rm $NGINX_LINK
fi

# Tạo lại symlink
sudo ln -s $NGINX_CONF $NGINX_LINK

echo "=== Test & reload Nginx ==="
sudo nginx -t || { echo "❌ Nginx config error"; exit 1; }
sudo systemctl reload nginx

echo "=== Check DNS before SSL ==="
RESOLVE=$(dig +short $DOMAIN)

if [ -z "$RESOLVE" ]; then
    echo "❌ DNS chưa trỏ, dừng cấp SSL"
    exit 1
fi

echo "=== Obtain SSL ==="
sudo certbot --nginx -d $DOMAIN \
  --non-interactive \
  --agree-tos \
  -m $EMAIL \
  --redirect

echo "=== DONE: https://$DOMAIN ==="
```

#### How to run scrip
```
nano setup-nginx.sh
chmod +x setup-nginx.sh
./setup-nginx.sh
```
