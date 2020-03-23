# 安装gitlab

1. 安装docker
2. 安装docker-compose
```
sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose -v
```
3. 安装gitlab
```
mkdir -p /home/data/gitlab/config
touch /home/data/gitlab/docker-compose.yml

gitlab:
    image: gitlab/gitlab-ce:11.3.6-ce.0
    restart: always
    hostname: '192.168.1.11'
    environment:
        GITLAB_OMNIBUS_CONFIG: |
            external_url 'https://192.168.1.11:8443'
            nginx['redirect_http_to_https'] = true
            letsencrypt['enable'] = false
            nginx['ssl_certificate'] = "/etc/gitlab/nginx.pem"
            nginx['ssl_certificate_key'] = "/etc/gitlab/nginx.key"
            # Add any other gitlab.rb configuration here, each on its own line
    ports:
        - 8443:8443
    volumes:
        - ./data:/var/opt/gitlab
        - ./logs:/var/log/gitlab
        - ./config:/etc/gitlab
        
  sudo openssl req -new -x509 -days 36500 -nodes -out config/nginx.pem \
         -keyout config/nginx.key -subj "/C=US/CN=gitlab/O=gitlab.com"
         
  docker-compose up
```
