
# iGotify

A small Gotify server notification assistent that decrypt the message and trigger a Push Notifications to iOS Devices via Apple's APNs with my service SecNtfy

SecNtfy will be available for public later this year.

Download Link to iGotify down below

&nbsp;
## ⭐ Features

* show received notifications with markdown
* decrypted the message with a public key that is generated from the iGotify device
* sending the decrypted message to SecNtfy and forwarded it to Apple's APN service, without saving the payload
* multiuser support

&nbsp;
## 🔧 How to Install Gotify & iGotify-Notification-Assist

### 🐳 Docker `docker-compose.yaml`

### Installation

1. Create a file with the name `docker-compose.yaml` or clone this repo and go to step 3
2. Please use the latest and recommended version of docker and docker compose
3. Copy the code down below in the yaml file
4. change environment variables in the compose file
5. execute `docker compose up -d` to start the docker compose

&nbsp;
### Needed environment variables

* `GOTIFY_DEFAULTUSER_PASS` = the user password for the defaultuser

### Optional environment variables

* `GOTIFY_URLS` = the local gotify sever URL e.g.: `http://gotify`
* `GOTIFY_CLIENT_TOKENS` = the client token from the Gotify Client e.g.: `cXXXXXXXX`
* `SECNTFY_TOKENS` = the SecNtfy Token that you get from the app after configure it e.g.: `NTFY-DEVICE-XXXXXX`

#### All these configuration can be found after configure the app. It will display it for you
#### Please note you can configure multiple instances of local gotify server by adding a semicolon `;` after each environment value e.g.:

* `GOTIFY_URLS: 'http://gotify;http://gotify2;http://gotify3;...'`
* `GOTIFY_CLIENT_TOKENS: 'cXXXXXXXX1;cXXXXXXXX2;cXXXXXXXX3;...'`
* `SECNTFY_TOKENS: 'NTFY-DEVICE-XXXXXX1;NTFY-DEVICE-XXXXXX2;NTFY-DEVICE-XXXXXX3;...'`

&nbsp;

```bash
version: '3.8'

services:
  gotify:
    container_name: gotify
    hostname: gotify
    image: gotify/server          # Uncommand correct server image
    # image: gotify/server-arm7
    # image: gotify/server-arm64
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    networks:
      - net
    ports:
      - "8680:80"
    volumes:
      - data:/app/data
    environment:
      GOTIFY_DEFAULTUSER_PASS:  'my-very-strong-password'   # Change me!!!!!

  igotify:
    container_name: igotify
    hostname: igotify
    image: ghcr.io/androidseb25/igotify-notification-assist:latest
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    pull_policy: always
    networks:
      - net
    ports:
      - "8681:8080"
    volumes:
      - api-data:/app/data
    #environment:                 # option environment see above note
    #  GOTIFY_URLS:          ''
    #  GOTIFY_CLIENT_TOKENS: ''
    #  SECNTFY_TOKENS:       ''

networks:
  net:

volumes:
  data:
  api-data:
```
*Thank you The_Think3r for the compose file and @herrpandora*

&nbsp;
### (Optional) NGINX Proxy Manager

When someone have problem's with incoming notifications on the app, please try this options under Advance Settings for the setuped proxies

```bash
proxy_set_header   Host $http_host;
proxy_connect_timeout   1m;
proxy_send_timeout      1m;
proxy_read_timeout      1m;
```

Also **don't** check the boxes which say "HTTP/2 Support" and "HSTS enabled".

*Thank you to @TBT-TBT for sharing this notice*

&nbsp;
### Traefik Config

```bash
version: "3.8"

services:
  gotify:
    container_name: gotify
    hostname: gotify
    image: gotify/server          # Uncommand correct server image
    # image: gotify/server-arm7
    # image: gotify/server-arm64
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    volumes:
      - data:/app/data
    environment:
      GOTIFY_DEFAULTUSER_PASS:  'my-very-strong-password'   # Change me!!!!!
      GOTIFY_REGISTRATION:       'false'
    labels:
      traefik.docker.network: proxy
      traefik.enable: "true"
      traefik.http.routers.gotify-secure.entrypoints: websecure
      traefik.http.routers.gotify-secure.middlewares: default@file
      traefik.http.routers.gotify-secure.rule: Host(`gotify.domain-name.de`)
      traefik.http.routers.gotify-secure.service: gotify
      traefik.http.routers.gotify-secure.tls: "true"
      traefik.http.routers.gotify-secure.tls.certresolver: http_resolver
      traefik.http.routers.gotify.entrypoints: web
      traefik.http.routers.gotify.rule: Host(`gotify.domain-name.de`)
      traefik.http.services.gotify.loadbalancer.server.port: "80"
    networks:
      default: null
      proxy: null
    volumes:
      - data:/app/data

  igotify-notification: # (iGotify-Notification-Assistent)
    container_name: igotify
    hostname: igotify
    image: ghcr.io/androidseb25/igotify-notification-assist:latest
    restart: always
    security_opt:
      - no-new-privileges:true
    pull_policy: always
    volumes:
      - api-data:/app/data
      
    #environment:                 # option environment see above note
    #  GOTIFY_URLS:          ''
    #  GOTIFY_CLIENT_TOKENS: ''
    #  SECNTFY_TOKENS:       ''
    
    labels:
      traefik.docker.network: proxy
      traefik.enable: "true"
      traefik.http.routers.igotify-secure.entrypoints: websecure
      traefik.http.routers.igotify-secure.middlewares: default@file
      traefik.http.routers.igotify-secure.rule: Host(`igotify.domain-name.de`)
      traefik.http.routers.igotify-secure.service: igotify
      traefik.http.routers.igotify-secure.tls: "true"
      traefik.http.routers.igotify-secure.tls.certresolver: http_resolver
      traefik.http.routers.igotify.entrypoints: web
      traefik.http.routers.igotify.rule: Host(`igotify.domain-name.de`)
      traefik.http.services.igotify.loadbalancer.server.port: "8080"
    networks:
      default: null
      proxy: null

networks:
  default:
  proxy:
    external: true
volumes:
  data:
  api-data:
```
*Thank you to @majo1989 for sharing this config*

&nbsp;
## 🔧 Install iGotify app

Download from [AppStore](https://apps.apple.com/de/app/igotify/id6473452512?itsct=apps_box_badge&amp;itscg=30200)

[![Download on the App Store](https://tools.applemediaservices.com/api/badges/download-on-the-app-store/black/de-de?size=350&amp;releaseDate=1702425600)](https://apps.apple.com/de/app/igotify/id6473452512?itsct=apps_box_badge&amp;itscg=30200)

For Bugs or feedback please send me a PM in Discord under the name sebakaderangler or fill out the issue formular here on GitHub.

On the login screen you need to enter the Gotify Server URL and the URL from the Notification Assist, if you use the URL with a port please enter it, too! (Image 1)

After the checks for the URL are finished and correct you need to login with your login credentials. (Image 2)

&nbsp;

![](https://raw.githubusercontent.com/androidseb25/iGotify-Notification-Assistent/main/login_screen_1.png)
![](https://raw.githubusercontent.com/androidseb25/iGotify-Notification-Assistent/main/login_screen_2.png)

&nbsp;
 
And if everythink is ok, you're logged in 🎉

Now you receive background notifications when Gotify receives a message.

## Translation
If you want to be a part of the translation team please create a issue:

**Title: Translation: *language***

**Description: crowdin username and why you would be a part of the translation team**

maybe you've been invited soon

The link to the crowdin project: [https://de.crowdin.com/project/igotify](https://de.crowdin.com/project/igotify)
