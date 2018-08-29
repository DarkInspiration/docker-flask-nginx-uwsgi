# Docker container for Flask, Nginx, and uWSGI, + Letsencrypt's certbot-auto for HTTPS

## Purpose:
Provide Dockerfile and all applicable config and base Flask scripts necessary to start a webpage, with a script to automate HTTPS re-configuration.

## Why do I use it?:
(Be sure docker is installed.  If not installed, run install_docker.sh) 

With this container and a built image (or pulling the image from ucnt/flaskwebpage), you can get an HTTP or HTTPS server setup in 1 commands
- HTTP: sudo docker run -d -p 80:80 -p 443:443 --restart=always -t --name flaskwebpage flaskwebpage 
- HTTPS (change parameters): sudo docker run -d -p 80:80 -p 443:443 --restart=always -t --name flaskwebpage flaskwebpage "-d example.com,www.example.com -n example.com -e my@email.com" 

Notes: 
- If you arleady setupt he server as HTTP and you want HTTPS, run: /home/flask/conf/setup-https.py -d [domain_list_csv] -n [certname] -e [email_address]
- You can access the container via: sudo docker exec -i -t flaskwebpage /bin/bash

## More thoughts:
https://www.mattsvensson.com/nerdings/2017/6/30/docker-flasknginxuwsgi

## Notes/Details:
- Folder/File Sctructure:
    All of the files+folders in this repo will be, by default, put into /home/flask.  If you modify this you need to update the Dockerfile.
    The /home/flask/app folder will contain the Flask app.  As long as the wsgi.py file uses "app" not "application," you can swap in and out any flask app that you want (so long as you have the necessary libraries installed).
  
  
  
  
## Services/Notes
- This script uses linux's Supervisor to monitor and control uWSGI and nginx.
- Port 443 is left on the run command in case you want to use it.  If you never will, you can remove "-p 443:443"
  

## HTTPS Setup Options (assumes 1 domain per container instance)
  
- <b>Easy way</b>: Put the domain info in the docker run command: 

sudo docker run -d -p 80:80 -p 443:443 --restart=always -t --name flaskwebpage flaskwebpage "-d example.com,www.example.com -n example.com -e my@email.com"

- <b>Semi-easy way</b>: Run the docker container for 443 as well as 80 then run the automated setup script after the container is up:

sudo docker run -d -p 80:80 -p 443:443 --restart=always -t --name flaskwebpage flaskwebpage

/home/flask/conf/setup-https.py -d example.com,www.example.com -n example.com -e my@email.com

- <b>HARD way</b>: 

Run: /home/flask/certbot-auto certonly -d [YOURDOMAIN] -w /home/flask/app
OR 
Copy your existing certs to the folder of your choice.  THEN...
      
    - Adjust /home/flask/conf/nginx-https-template.conf to use HTTPS by replacing YOURDOMAIN with the domain you are setting up and, if you copied a cert into a folder, change the directory from /etc/letsencrpyt/live
      
    - Remove /etc/nginx/sites-enabled/nginx-http.conf
      
    - Re-link ntinx-https.conf to /etc/nginx/sites-enabled: ln -s /home/flask/conf/nginx-https-template.conf /etc/nginx/sites-enabled/nginx-http.conf
      
    - Restart the supervisor service
    
  
  
## Credits
Credit to Thatcher Peskens (https://github.com/atupal/dockerfile.flask-uwsgi-nginx), who this code was forked from.
    




