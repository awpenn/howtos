## Installing Jupyterhub ( not tljh ) on Centos ( AWS )
- can't install tljh on centos without major headache, so just went with regular jupyterhub

- install `jupyterhub` with: https://jupyterhub.readthedocs.io/en/stable/quickstart.html#installation

- had to add `:/usr/local/bin` to end of PATH in root user's .bash_profile, and then make the `secure_path` variable line like so in `/etc/sudoers`
```
Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin
```
otherwise jupyterhub command didnt work without sourcing root's .bash_profile every login

- created dir `/var/run` and added `jupyterhub.service` file like so
```
[Service]
Environment=PATH=/opt/miniconda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
ExecStart=/usr/local/bin/jupyterhub --port 8080 ##this will be different!!!
Restart=on-failure
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=jupyterhub

[Install]
WantedBy=multi-user.target
```
- added symlink and made systemd aware of service
```
# Create a symbolic link into the systemd directory
$ ln -s /var/run/jupyterhub.service /etc/systemd/system/jupyterhub.service

# Make the systemd aware of the service
$ systemctl daemon-reload
```
- should then start up with `sudo service jupyterhub start`

## Proxy setup
- in jupyterhub config ( eg. `/etc/jupyterhub/jupyterhub_config.py` )
    - enable `c.JupyterHub.base_url` variable and set URL, eg. 
    ```
    c.JupyterHub.bind_url = 'http://:8080/dbscripts/'
    ```
    - restart jupyterhub service

- server:
    - get OV to open the necessary port
    - at `/etc/httpd/conf.d` add a .conf file, eg. `dbscripts.conf` with the following: ##7/30/21 this part doesnt work yet
    ```
    <Proxy *>
    Allow from localhost
    </Proxy>

    <VirtualHost *:80>
    #JupyterHub
    RewriteEngine On
    RewriteCond %{HTTP:Connection} Upgrade [NC]
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteRule /dbscripts/(.*) ws://127.0.0.1:8080/dbscripts/$1 [P,L]
    RewriteRule /dbscripts/(.*) http://127.0.0.1:8080/dbscripts/$1 [P,L]
    
    #Preserve Host header to avoid cross-origin problems
    ProxyPreserveHost On
    #proxy to JupyterHub
    ProxyPass /dbscripts/ http://127.0.0.1:8080/dbscripts
    ProxyPassReverse /dbscripts/  http://127.0.0.1:8080/dbscripts

    </VirtualHost>
    ```
    - restart httpd
    ```
    service httpd restart
    ```

## user setup
- without fudging around a bunch, I couldnt enable create user in the admin panel, and login is based on users in the linux user info, so for each, need to create user and pass in command line

```
sudo adduser juser-adspid
sudo passwd ( will prompt for password )
```

- imported the apps at the same directory level as users.
    - so notebook run command like: `%run -i '../id_db/scripts/main_menu.py'`
    - .env DBIP value is `localhost`