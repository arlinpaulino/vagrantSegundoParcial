Vagrant.configure("2") do |config|
    # Configuración global
    config.vm.box = "ubuntu/focal64"

    # Configuración del primer servidor Apache
    config.vm.define "apache1" do |apache1|
        apache1.vm.network "private_network", ip: "192.168.56.4"
        apache1.vm.hostname = "apache1"
        apache1.vm.provision "shell", inline: <<-SHELL
            sudo apt-get update
            sudo apt-get install -y apache2
            sudo systemctl enable apache2
            echo "<h1>Servidor Apache 1</h1>" | sudo tee /var/www/html/index.html
        SHELL

        # Compartir un directorio local con el directorio web de Apache
        apache1.vm.synced_folder "./web_apache1", "/var/www/html"
    end

    # Configuración del segundo servidor Apache
    config.vm.define "apache2" do |apache2|
        apache2.vm.network "private_network", ip: "192.168.56.5"
        apache2.vm.hostname = "apache2"
        apache2.vm.provision "shell", inline: <<-SHELL
            sudo apt-get update
            sudo apt-get install -y apache2
            sudo systemctl enable apache2
            echo "<h1>Servidor Apache 2</h1>" | sudo tee /var/www/html/index.html
        SHELL

        # Compartir un directorio local diferente con el directorio web de Apache
        apache2.vm.synced_folder "./web_apache2", "/var/www/html"
    end

    # Configuración del servidor Nginx como balanceador de carca
    config.vm.define "nginx" do |nginx|
        nginx.vm.network "private_network", ip: "192.168.56.6"
        nginx.vm.hostname = "nginx"
        nginx.vm.provision "shell", inline: <<-SHELL
            sudo apt-get update
            sudo apt-get install -y nginx

            cat <<EOF | sudo tee /etc/nginx/sites-available/default
upstream backend {
    server 192.168.56.4;
    server 192.168.56.5;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;   
    }
}
EOF

            sudo nginx -t
            sudo systemctl restart nginx
        SHELL
    end
end

