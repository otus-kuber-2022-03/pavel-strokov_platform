FROM    nginx:alpine
ENV TZ=Europe/Moscow
ARG UID=1001
RUN addgroup -g $UID webuser && adduser -u $UID -G webuser -s /bin/sh -D webuser
COPY  ./etc/nginx.conf /etc/nginx/conf.d/default.conf
COPY  ./public /app
EXPOSE 8000