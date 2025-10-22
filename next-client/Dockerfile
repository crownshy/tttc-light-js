FROM node:24

EXPOSE 3000
RUN mkdir /app
WORKDIR /app
COPY common ./common
COPY next-client ./next-client

WORKDIR /app/common

RUN npm install
RUN npm run build 

WORKDIR /app/next-client
RUN npm install
RUN npm run build 

CMD ["npm", "run", "start"]


