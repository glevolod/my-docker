FROM nginx:alpine
ARG HOST
WORKDIR /var/www
COPY sites/site.conf /etc/nginx/sites-available/
RUN sed -i "s/__server_name__/${HOST}/g" /etc/nginx/sites-available/site.conf
CMD ["nginx"]
EXPOSE 80 443