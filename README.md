# Ansible LEMP Installation for Ubuntu 20.04

This configuration is tailored for Ubuntu 20.04 and Ansible version [core 2.15.4]. It includes essential components such as Nginx, PHP 8.2, MySQL, and phpMyAdmin. The Nginx configurations provided cater to use cases involving React and Django applications. Furthermore, the setup is highly customizable, allowing you to adapt it to your specific project needs.

## Prerequisites

- [Ansible](https://www.ansible.com/) installed on your local machine. You can install Ansible using your operating system's package manager.

## Features
## Automated LEMP Stack Installation:

Deploy a complete LEMP (Linux, Nginx, MySQL, PHP) stack on your local machine with a single Ansible command.

## Customizable Configuration:
Modify variables in the Ansible playbook to customize Nginx, MySQL, and PHP configurations according to your specific project requirements.

## Version Compatibility:
Ensure compatibility with different versions of Nginx, MySQL, and PHP. Easily update version numbers in the configuration to match your project's needs.
## Project Structure

```bash
.
├── ansible.cfg
├── inventroy
├── lemp-server
│   ├── README.md
│   ├── defaults
│   │   └── main.yml
│   ├── files
│   ├── handlers
│   │   └── main.yml
│   ├── meta
│   │   └── main.yml
│   ├── tasks
│   │   └── main.yml
│   ├── templates
│   │   └── server.conf
│   ├── tests
│   │   └── test.yml
│   └── vars
│       └── main.yml
└── lemp-server.yaml
10 directories, 11 files
```


## Setup

Clone the Repository:
```bash
git clone https://github.com/sabarishOfficial/ansible-lemp-installation.git
cd ansible-lemp-installation
```
### Nginx Configurations
Below is an example Nginx server block configuration catering to React and Django use cases. If you need to customize your configuration, modify the server.conf file in the templates directory.
```bash
# Nginx Server Block Configuration
server {
    # Server Name
    server_name _;

    # Client Max Body Size
    client_max_body_size 100m;

    # React Use Case
    location / {
        root /var/www/html/frontend;
        try_files $uri /index.html;
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Origin' '*' always;
    }

    # Django Use Case
    location /api/ {
        # Handle OPTIONS request for CORS
        if ($request_method = 'OPTIONS') {
            add_header 'Access-Control-Allow-Origin' '*' always;
            add_header 'Access-Control-Allow-Methods' '*' always;
            add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range,*' always;
            add_header 'Access-Control-Allow-Credentials' 'true';
            add_header 'Content-Length' 0;
            add_header 'Content-Type' 'text/plain charset=UTF-8';
            return 200;
        }

        # Proxy requests to the upstream server
        alias /var/www/html/backend/;
        proxy_pass http://127.0.0.1:8000;

        # Proxy headers if needed
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # Configure proxy timeouts if needed
        proxy_connect_timeout 60s;
        proxy_read_timeout 60s;

        # CORS Headers
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Origin' '*' always;
    }

    # phpMyAdmin Configuration
    location /phpmyadmin {
        root /usr/share/;
        index index.php;
        try_files $uri $uri/ =404;

        # Deny access to specific phpMyAdmin paths
        location ~ ^/phpmyadmin/(doc|sql|setup)/ {
            deny all;
        }

        # Pass PHP requests to php-fpm
        location ~ /phpmyadmin/(.+\.php)$ {
            fastcgi_pass unix:/run/php/php8.2-fpm.sock;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi.conf;
            include snippets/fastcgi-php.conf;
            proxy_read_timeout 600s;
        }
    }
}
```
## Configure Inventory:

Update the inventory file with your server details
```bash
server ansible_host=IPADDRESS ansible_user=Linux_username ansible_ssh_private_key_file=example.pem
```
## Check Ansible Playbook Syntax
```bash
ansible-playbook lemp-server.yaml --syntax-check
```
## Test Server Connection
```bash
ansible -i inventory -m ping server
```
You should receive a successful ping response

## Run Ansible Playbook:
```bash
ansible-playbook -i inventory lemp-server.yaml
```
