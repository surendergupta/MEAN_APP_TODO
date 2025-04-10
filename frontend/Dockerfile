FROM node:18-alpine AS buider
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build --prod

FROM nginx:alpine
COPY --from=buider /app/dist/angular-15-crud /usr/share/nginx/html

EXPOSE 80

CMD [ "nginx", "-g", "daemon off;" ]