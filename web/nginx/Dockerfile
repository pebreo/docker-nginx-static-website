FROM nginx
COPY ./django/nginx.conf /etc/nginx/nginx.conf

#FROM tutum/nginx
#RUN rm /etc/nginx/sites-enabled/default
#RUN sed -i.bak "s/access_log \/var\/log\/nginx\/access.log;/\n      access_log syslog:server=loggly;\n/g" ./etc/nginx/nginx.conf
#RUN sed -i.bak "s/error_log \/var\/log\/nginx\/error.log;/\n      error_log syslog:server=loggly;\n/g" /etc/nginx/nginx.conf
#ADD sites-enabled/ /etc/nginx/sites-enabled

