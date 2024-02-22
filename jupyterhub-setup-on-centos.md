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
    - enable `c.JupyterHub.base_url` variable and set URL, eg. !<--- say base here but I think mean bind, base is deprecated>
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


## Custom html stuff
- edit files at `/usr/local/share/jupyterhub/templates/`


## 8/9/21 fix
- if the jupyterhub and proxy starts correctly, that last thing you'll see is something like `now running at http://URL`
    - if something didnt work right and you keep getting `404 Not Found` from `/` , need to edit `etc/hosts`
    - check hostname with `hostname` command in CL
    - add something like `127.0.0.1 kentaurus kentaurus.lisanwanglab.org` to hosts

## 3/35/22 notes following service failure following server/machine reboot
- moved `jupyterhub.service` from `/var/run` because this directory is meant for temporary files and gets cleared on reboot
    - moved it to `/etc/systemd/system/` so no need for symlink
- now service runs, but attempting to reach `url/dbscripts` results in some kind of redirect timeout
- tried changing `jupyterhub.service` `ExecStart` line from
```
ExecStart=/usr/local/bin/jupyterhub --port 8080 ##this will be different!!!
```
to 
```
ExecStart=/usr/local/bin/jupyterhub --config=/etc/jupyterhub/jupyterhub_config.py --port 8080 ##this will be different!!!
```
which maybe changed something in that now trying to access /dbscripts redirects in address bar to /hub/dbscripts, but I can't remember if that was happening previously 

- as per OV, I changed the reference to `/dbscripts` in `jupyterhub_config.py` to use `c.JupyterHub.base_url` instead of `bind_url`, this worked, but need to figure out my `bind_url` doesnt work because `base_url` is deprecated

## 080822 Cross-origin error jupyterhub behind teleport
- probably has to do with now the url is `jhub-kentaurus.gate.[etc]`
``` 
`journalctl -xe`
Blocking Cross Origin API request for /dbscripts/user/juser-cdb/api/contents/main_menu.ipynb.  Origin: https://jhub-kentaurus.gate.lisanwanglab.org, Host: localhost:8080
```
- FIX: had to add `c.Spawner.args = ['--NotebookApp.allow_origin=https://jhub-kentaurus.gate.lisanwanglab.org']` to `jupyterhub_config.py` at `/etc/jupyterhub`