# Docker/compose template for my basic Django projects
Sample project to deploy/serve a Django application using Docker, including a PostgreSQL database, and static/media files serving

# Nginx/Proxy and SSL
This is intended to use with nginx-proxy and its companion, to auto generate HTTPS certificates.

# First steps
### Environment variables are set on a `.env` file to be used with Python Decouple https://github.com/henriquebastos/python-decouple/ or similar.

1) Copy this project files to your django project root
2) Rename `.env.example` to `.env` and edit at least the first two lines r
3) Run steps below 


# Requirements on a EC2 instance (Amazon linux):
### Run the following commands
Amazon Linux installations:
```
# Install Docker
sudo yum update -y
sudo amazon-linux-extras install docker git
sudo service docker start
sudo usermod -a -G docker ec2-user

# Install docker-compose:
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Install git:
sudo yum install -y git
```


# Set up Nginx proxy and companion

### Please set your email to use with the Let's Encrypt certificates
```
DEFAULT_LETSENCRYPT_EMAIL=<INSERT EMAIL HERE>
```

### Now run the nginx and companion dockers:
```
# Create shared network between containers
docker network create nginx-gateway

# Run Nginx Proxy
docker run --detach \
    --name nginx-proxy \
    --publish 80:80 \
    --publish 443:443 \
    --volume certs:/etc/nginx/certs \
    --volume vhost:/etc/nginx/vhost.d \
    --volume html:/usr/share/nginx/html \
    --volume /var/run/docker.sock:/tmp/docker.sock:ro \
    --net nginx-gateway nginxproxy/nginx-proxy


# UP companion
docker run --detach \
    --name nginx-proxy-acme \
    --volumes-from nginx-proxy \
    --volume /var/run/docker.sock:/var/run/docker.sock:ro \
    --volume acme:/etc/acme.sh \
    --env "DEFAULT_EMAIL=${DEFAULT_LETSENCRYPT_EMAIL}" \
    --net nginx-gateway nginxproxy/acme-companion
```


Build and run docker compose
```
docker-compose up -d --build
```

## References:
- https://luis-sena.medium.com/creating-the-perfect-python-dockerfile-51bdec41f1c8

# TODO:
- Add memory log limit (@luzfcb at https://github.com/djangopackages/djangopackages/pull/583/files)
- Acknowledge this article:
https://luis-sena.medium.com/creating-the-perfect-python-dockerfile-51bdec41f1c8
- Consider this article: https://betterprogramming.pub/faster-python-in-docker-d1a71a9b9917
- Read this: https://luis-sena.medium.com/gunicorn-worker-types-youre-probably-using-them-wrong-381239e13594
- Include django and python packages:
    - python decouple 
    - django-extensions
    - sentry-sdk
    - django-allauth 
    - django-s3-storage
    - django-debug-toolbar
    - Honorable mentions:
        - Django rest framework
            - Django cors headers
            - django-filter
        - psycopg2-binary
        - django-autocomplete-light
        - django-querysetsequence 
        - sorl-thumbnail
        - celery, celery beat
        - zappa (amazon lambda)
        - black
    - Non python:
        - CircleCI config
    


  
