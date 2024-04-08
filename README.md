## 1.0 On your terminal
```sh
$ sudo apt-get update
```
```sh
$ sudo apt-get install python3-pip python3-dev libpq-dev nginx
```
Upgrading pip && installing virtualenv
```sh
$ sudo -H pip3 install --upgrade pip
```
```sh
$ sudo -H pip3 install virtualenv
```
Creating and Activating VirtualEnviroment
```sh
$ virtualenv myprojectenv
```
```sh
$ source myprojectenv/bin/activate
```
## 2.0 Installing gunicorn

```sh
$ pip install django gunicorn psycopg2
```

# 3.0 Adjust settings.py file, ALLOWED_HOSTS
```sh
$ nano settings.py
```

Trying gunicorn on port 8000
```sh
$ sudo ufw allow 8000
```
```sh
$ cd ~/myproject
```
```sh
$ gunicorn --bind 0.0.0.0:8000 myproject.wsgi
```

Deativate
```sh
$ deactivate
```
# 4.0 Create gunicorn servicemd file
```sh
$ sudo nano /etc/systemd/system/gunicorn.service
```
```sh
[Unit]
Description=gunicorn daemon
#Requires=gunicorn.socket
After=network.target

[Service]
User=username
Group=www-data
WorkingDirectory=/home/username/myproject

#Path to Gunicorn, Use command $ whereis gunicorn, to locate gunicorn
ExecStart=/home/username/venv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/home/username/myproject/myproject.sock myproject.wsgi:application

[Install]
WantedBy=multi-user.target
```
# 5.0 Check for gunicorn status and socket file
```sh
$ sudo systemctl status gunicorn
``
```sh
$ cd myprject && ls
## you should see myprojecct.socket

Output
manage.py  myproject  myprojectenv  myproject.sock  static
```
If Gunicorn ran into problems, run the command bello to see error logs 
```sh
$ sudo journalctl -u gunicorn
```
The project files are owned by the root user instead of a sudo user
The Working Directory path within the /etc/systemd/system/gunicorn.service file
does not point to the project directory
The configuration options given to the gunicorn process in the
ExecStart directive are not correct.
Check the following items: 

	1.The path to the gunicorn binary points to the actual location of the 
	  binary within the virtual environment
# 6.0 To LOCATE GUNICORN in LInux(I am using Ubuntu18),
Run 
```sh
$ where is gunicorn
```
	2. The --bind directive defines a file to create within
	  #a directory that Gunicorn can access

	3The myproject.wsgi:application is an accurate path to the WSGI callable.
	This means that when you’re in the WorkingDirectory, you should be able
	to reach the callable named application by looking in the myproject.wsgi module
	(which translates to a file called ./myproject/wsgi.py)

If you make changes to  /etc/systemd/system/gunicorn.service file,
reload the daemon to reread the service definition and restart
the Gunicorn process by typing:
```sh
$ sudo systemctl daemon-reload
$ sudo systemctl restart gunicorn
```
# 7.0 Configure Nginx to Proxy Pass to Gunicorn

Start by creating and opening a new server block in Nginx’s sites-available directory:
```sh
$ sudo nano /etc/nginx/sites-available/myproject
``
=======================================================================
```sh
server {
    listen 80;
    server_name server_domain_or_IP;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/username/myproject;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/username/myproject/myproject.sock;
    }
}

```
Now, we can enable the file by linking it to the sites-enabled directory:

```sh
$ sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled
```
# 8.0 Test you Nginx Configuration

```sh
$ sudo nginx -t
```
If no errors are reported, go ahead and restary Nginx
```sh
$ sudo systemctl restart nginx
```
Finally, we need to open up our firewall to normal traffic on port 80. Delete port 8000

```sh
$ sudo ufw delete allow 8000
$ sudo ufw allow 'Nginx Full'
```
You should now be able to go to your server’s domain or IP address
to view your application.
