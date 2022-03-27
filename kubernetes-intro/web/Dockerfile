FROM nginx:alpine
ENV TZ=Europe/Moscow
ARG UID=1001
ARG GID=1001
ENV UID=${UID}
ENV GID=${GID}
COPY  ./etc/nginx.conf /etc/nginx/conf.d/default.conf
COPY  ./public /app
RUN addgroup -g ${UID} www-user && adduser -u ${UID} -G www-user -s /bin/sh -D www-user && chown -R www-user:www-user /app
#USER ${UID}
EXPOSE 8000