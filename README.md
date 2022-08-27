# DMLAPI

First we will start from downloading / pulling docker image for our api, please type in this command in your terminal to pull this image
```console
docker pull nachikamod/dmlapi:0.0.1
```

To create a api container from given image use following command

```console
docker run -d -p 3000:3000 --restart unless-stopped nachikamod/dmlapi:0.0.1
```

Now, our api is running on 0.0.0.0:3000, no further configuration is required for the api container

Now, deploying web app is little bit of tricky part since we are using angular for frontend. Since angular compiles our source into web application it basically replaces environment varibale on build with plain text. So there is no way we can change environment variable  during runtime.

So, whenever the ip or domain of our endpoint (api server) changes we will have to create a new container for the web application. (Atleast that's what my knowledge limitations says).

We will start from editing our Dockerfile in the root of our project, which looks like this

```Dockerfile
FROM node:latest as builder

#stage 1
WORKDIR /app
COPY . .

RUN npm install @angular/cli@latest -g

RUN npm install --force

ENV API_URL="http://dmlapi.ddns.net:3000" # replace this api url / endpoint with your ip or domain

RUN ng config -g cli.warnings.versionMismatch false
RUN ng build

#stage 2
FROM nginx:alpine

VOLUME  /var/cache/nginx

COPY --from=builder /app/dist/dmlapi /usr/share/nginx/html
COPY --from=builder /app/.docker/.config/nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 4200
```

Once done with the editing, execute this command to create a image of a container

```Console
dokcer build -t nachikamod/dmlweb:0.0.1 .
# You can change the image, by replacing required name in place of <name>
docker build -t <name>:0.0.1 .
```

It will take some time, once done, execute following command replacing `nachikamod/dmlweb` with your required name you have changed earlier


```console
docker run -d -p 4200:4200 --restart unless-stopped nachikamod/dmlweb:0.0.1
```

Now here our setup is done,
we can now access our api from `0.0.0.0:3000` or `localhost:3000` or `<local_ip>:3000` or `<public_ip>:3000` (Only if you have exposed port 3000 to your internet).

The web application is accessible from `0.0.0.0:4200` or `localhost:4200` or `<local_ip>:4200` or `<public_ip>:4200` (Only if you have exposed port 4200 to your internet).

Create an account on web application and start uploading your model.
Make sure you provide requirements.txt generated using `pip freeze > requirements.txt`.

For the mobile application, you can change api url using github action secrets
or you can change it directly from the source code and build application locally.
