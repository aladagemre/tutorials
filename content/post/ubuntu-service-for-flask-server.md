+++
tags = ["python", "ubuntu"]
Description = "Creating Ubuntu Service for Flask server"
date = "2015-12-27T19:09:21+02:00"
menu = "main"
title = "Creating Ubuntu Service for Flask server"
Summary = "Creating Ubuntu Service for Flask server"
+++


Let's say you have a flask app and you want to create an Ubuntu service for this app. This way, the service will restart whenever the server restarts or the service fails. Here is a sample flask app on **/home/user/server.py**:

    #!/usr/bin/python
    from flask import Flask, request
    app = Flask(__name__)
    @app.route('/', methods=['GET'])
    def index():
        return "Hello world"

    if __name__ == '__main__':
       app.run(host="0.0.0.0", port=31013)


We have to write the following service configuration in **/etc/init/flask.conf**:

    description "flask"
    start on stopped rc RUNLEVEL=[2345]
    respawn
    exec python /home/user/server.py


Try **sudo service flask start**. If you receive any error, you may have to create an additional file **/lib/systemd/system/flask.service**

    [Unit]
    Description=Flask web server

    [Install]
    WantedBy=multi-user.target

    [Service]
    User=root
    Group=root
    PermissionsStartOnly=true
    ExecStart=/home/user/server.py
    TimeoutSec=600
    Restart=on-failure
    RuntimeDirectoryMode=755


You now can **sudo service flask start/stop/status**.
