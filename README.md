# Building Flask container image
This project provides a Dockerfile and a configuration information for building Flask container image.

## 1. Environment
```
  Ubuntu 18.04.4 LTS
  Docker 18.06.3-ce
```
## 2. Docker DNS configuaration
It is necessary to setup Docker default DNS.
While building a container image, I cannot install flask by ```pip install flask``` becasue of DNS issue. 
The below configuration helpful to resolve this issue.

NOTE - I tried to create ```etc/docker/daemon.json``` and add ```{"dns":["MY_DNS1","Y_DNS1"]}```, but restarting docker was failed with the error, "dns configuration failure".

### 2.1 Create override.conf
```
$ vim /etc/systemd/system/docker.service.d/override.conf
```

### 2.2 Add the below content
It is needed to add an empty "ExecStart=" to 'clear'.
```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --dns [ADD_YOUR_DNS] -H fd://
```

### 2.3 Restart Docker
```
sudo systemctl daemon-reload
sudo systemctl stop docker
sudo systemctl start docker

```

## 3. Build a Flask container image
### 3.1 Create an app.py for testing
```
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"

if __name__=='__main__':
    app.run(debug=True,host='0.0.0.0')
```

### 3.2 Create a Dockerfile
```
FROM python
COPY . /app
WORKDIR /app
RUN pip install flask
EXPOSE 5000
CMD ["python", "app.py"]
```

### 3.3 Build image and run
```
$ docker build -t flask-test .
$ docker images
$ docker run -d -p 5000:5000 flask-test
$ docker ps
```

### 3.4 Check server operation
Access ```http://127.0.0.1:5000``` and check if it is running.
