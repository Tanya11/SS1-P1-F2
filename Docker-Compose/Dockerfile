
FROM node
WORKDIR /app
ADD . /app
COPY package.json .
RUN npm install --quiet
ENV PORT 3000
ENV IP "172.16.1.4"
