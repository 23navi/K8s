FROM node:14-alpine

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 80

CMD [ "node", "auth-app.js" ]


## Image name: navi23/k8-networking-auth-api