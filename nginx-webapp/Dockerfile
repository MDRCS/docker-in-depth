FROM nginx:latest

ADD index.html /www/
ADD nginx.conf /www/nginx/conf.d/default.conf

#EXPOSE 80
#EXPOSE 443

CMD nginx -g "daemon off;"
