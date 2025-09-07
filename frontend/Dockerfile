FROM node:18 AS build

# set work directory
WORKDIR /app

# install dependencies
COPY package*.json .
RUN npm i

# copy source
COPY . .

# build app
RUN npm run build


FROM nginx:1.23.2-alpine

# copy nginx config
COPY /.nginx/nginx.conf /etc/nginx/conf.d/default.conf

# remove default stuff
RUN rm -rf /usr/share/nginx/html/*

# copy build
COPY --from=build /app/build /usr/share/nginx/html

# run nginx as non-root
RUN touch /var/run/nginx.pid
RUN chown -R nginx:nginx /var/run/nginx.pid /usr/share/nginx/html /var/cache/nginx /var/log/nginx /etc/nginx/conf.d
USER nginx

# expose port
EXPOSE 3000

# run app
ENTRYPOINT ["nginx", "-g", "daemon off;"]
