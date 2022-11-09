# Docker
## Docker installieren
in Amazon AWS Linux

    sudo yum install docker -y
    
in Ubuntu Linux

    sudo pacman install docker -y
    
## Docker starten

    sudo systemctl enable docker
    sudo systemctl start docker
    
## Hello World aufrufen

    sudo docker run hello-world
    
## nginx Webserver in einen Docker Container packen und starten

    sudo docker build -t nginx-beispiel .
    sudo docker run --name nginx-beispiel-container -d -p 8081:80 nginx-beispiel
    
## Docker-Compose in Ubuntu Linux installieren

    sudo pacman -S docker-compose
    
## WordPress installieren

in einem Verzeichnis die Datei docker-compose.yaml erstellen und folgenden Inhalt einfügen:

     
    version: '2.3'
    services:
      nginx-proxy:
        image: jwilder/nginx-proxy
        restart: always
        volumes:
          - ./nginx-proxy/nginx-proxy-certs:/etc/nginx/certs:ro
          - ./nginx-proxy/nginx-proxy-vhost:/etc/nginx/vhost.d
          - /usr/share/nginx/html
          - /var/run/docker.sock:/tmp/docker.sock:ro
        ports:
          - 80:80
          - 443:443
        labels:
          - "com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy"
        networks:
          wp-network:
      nginx-letsencrypt:
        image: jrcs/letsencrypt-nginx-proxy-companion
        restart: always
        volumes:
          - ./nginx-proxy/nginx-proxy-certs:/etc/nginx/certs:rw
          - /var/run/docker.sock:/var/run/docker.sock:ro
        volumes_from:
          - nginx-proxy
        networks:
          wp-network:
      wpdb:
        image: mariadb:10.5.4
        restart: always
        environment:
          - MYSQL_ROOT_PASSWORD=db_passwd                     # change this
          - MYSQL_DATABASE=wordpress                          # change this
          - MYSQL_USER=wpuser                                 # change this
          - MYSQL_PASSWORD=wp_passwd                          # change this
        volumes:
          - ./wordpress-db/wpdb-data:/var/lib/mysql
        networks:
          wp-network:
      wp:
        depends_on:
          - wpdb
        image: wordpress:latest
        volumes:
          - ./wordpress/wp-data:/var/www/html/wordpress
        ports:
          - "8080:80"
        restart: always
        environment:
          - WORDPRESS_DB_HOST=wpdb:3306
          - WORDPRESS_DB_USER=wpuser                          # change this
          - WORDPRESS_DB_PASSWORD=wp_passwd                   # change this
          - WORDPRESS_DB_NAME=wordpress                       # change this
          - VIRTUAL_HOST=www.example.com                      # change this
          - VIRTUAL_PORT=80
          - VIRTUAL_PROTO=http
          - LETSENCRYPT_HOST=www.example.com                  # change this
          - LETSENCRYPT_EMAIL=your-mail@example.com           # change this
          - RESOLVE_TO_PROXY_IP=true
        networks:
          wp-network:
    networks:
      wp-network:
        driver: bridge
    volumes:
      nginx-proxy-certs:
      nginx-proxy-vhost:
      nginx-proxy-certs:
      wpdb-data:
      wp-data:
      
bei VIRTUAL_HOST und LETSENCRYPT_HOST die IP-Adresse des Computers, auf dem der Docker läuft, einfügen

bei LETSENCRYPT_EMAIL eine E-Mail-Adresse einfügen

Die Datei docker-compose.yaml speichern

im Verzeichnis, in dem die Datei docker-compose.yaml ist, folgenden Befehl ausführen:

    sudo docker-compose up -d

    
Folgende Adresse im Browser aufrufen:

    http://localhost:8080
