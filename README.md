# candy-dist

## Windows => Linux 배포

```ps1
# dist.ps1
$DistHost = "Set your own server host name"

git clone "https://github.com/rollrat/candy-web"
git clone "https://github.com/rollrat/candy-server"
git clone "https://github.com/rollrat/candy-ws"

cd candy-web
yarn build
Compress-Archive -Path build -DestinationPath dist-web.zip -Force 
cd ../candy-server
yarn build
Copy-Item -Path package.json -Destination dist/package.json
Copy-Item -Path package-lock.json -Destination dist/package-lock.json
Compress-Archive -Path dist -DestinationPath dist-s.zip -Force 
cd ../candy-ws
yarn build
Copy-Item -Path package.json -Destination dist/package.json
Copy-Item -Path package-lock.json -Destination dist/package-lock.json
Compress-Archive -Path dist -DestinationPath dist-ws.zip -Force 
cd ../

if (Test-Path dist-web.zip) {
  Remove-Item dist-web.zip
}
if (Test-Path dist-s.zip) {
  Remove-Item dist-s.zip
}
if (Test-Path dist-ws.zip) {
  Remove-Item dist-ws.zip
}

Move-Item -Path candy-web/dist-web.zip -Destination ./dist-web.zip
Move-Item -Path candy-server/dist-s.zip -Destination ./dist-s.zip
Move-Item -Path candy-ws/dist-ws.zip -Destination ./dist-ws.zip

scp dist-web.zip rollrat@${DistHost}:~/
scp dist-ws.zip rollrat@${DistHost}:~/
scp dist-s.zip rollrat@${DistHost}:~/

ssh rollrat@${DistHost} sudo apt install unzip nodejs npm
ssh rollrat@${DistHost} "npm i -g pm2"

ssh rollrat@${DistHost} sudo rm -rf web
ssh rollrat@${DistHost} sudo rm -rf ws
ssh rollrat@${DistHost} sudo rm -rf s

ssh rollrat@${DistHost} unzip dist-web.zip -d web
ssh rollrat@${DistHost} unzip dist-ws.zip -d ws
ssh rollrat@${DistHost} unzip dist-s.zip -d s

ssh rollrat@${DistHost} "cd s/dist && mkdir private"
scp candy-server/private/config.json rollrat@${DistHost}:~/s/dist/private/

ssh rollrat@${DistHost} "chmod -R 755 web/*"
ssh rollrat@${DistHost} "cd ws/dist && npm i --only=prod && sudo pm2 start app.js -f"
ssh rollrat@${DistHost} "cd s/dist && npm i --only=prod && sudo pm2 start app.js -f"
```

```nginx
        location / {
                # web server (front)
                proxy_pass http://0.0.0.0:4004;
        }

        location /api {
                # api server (backend)
                proxy_set_header X-Forwarded-For $remote_addr;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header Host $http_host;
                proxy_pass http://0.0.0.0:8864;
        }
```

## Linux => Linux 배포
