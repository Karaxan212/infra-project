FROM nginx:1.27-alpine
COPY index.html /usr/share/nginx/html/
EXPOSE 80
HEALTHCHECK CMD wget -qO- http://localhost/ || exit 1
