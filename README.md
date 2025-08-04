# WordPress Template

## Setup Script

```
#!/bin/bash

echo "WordPress Setup Script"

read -p "App Name: " app
read -p "App Domain: " domain
read -p "Lets Encrypt Email: " ssl_email

if [[ -n "$app" && -n "$domain" ]]
then
  echo "Creating dokku app $app..."
  dokku apps:create $app

  echo "Setting domain $domain..."
  dokku domains:set $app $domain

  echo "Setting up persistent storage..."
  dokku storage:ensure-directory $app
  sudo mkdir -p /var/lib/dokku/data/storage/${app}/wp-content
  sudo mkdir -p /var/lib/dokku/data/storage/${app}/wp-content/plugins
  sudo mkdir -p /var/lib/dokku/data/storage/${app}/wp-content/themes
  sudo chown -R 32767:32767 /var/lib/dokku/data/storage/${app}
  dokku storage:mount $app /var/lib/dokku/data/storage/$app/wp-content:/app/web/wp-content

  echo "Creating the database..."
  dokku mariadb:create $app-db --memory 2048 -C "TZ=Europe/Berlin"

  echo "Linking the database..."
  dokku mariadb:link $app-db $app

  echo "Setting environment variables..."
  dokku config:set $app WEB_HOST=https://$domain --no-restart
  curl -sS "https://api.wordpress.org/secret-key/1.1/salt/" | awk -F"'" '{printf "WP_%s=\047%s\047 ", $2, $4}' | xargs dokku config:set $app --no-restart


  echo "Setting up Lets encrypt..."
  dokku letsencrypt:set $app email $ssl_email
  dokku letsencrypt:enable $app

  echo "Finished WordPress setup!"
fi
```
