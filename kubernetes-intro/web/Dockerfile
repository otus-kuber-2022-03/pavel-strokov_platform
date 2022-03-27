FROM nginx:alpine
ENV TZ=Europe/Moscow
ARG UID=1001
RUN addgroup -g $UID www && adduser -u $UID -G www -s /bin/sh -D www
COPY  ./etc/nginx.conf /etc/nginx/conf.d/default.conf
COPY  ./public /app
RUN chown -R www:www /app
#USER $UID
EXPOSE 8000