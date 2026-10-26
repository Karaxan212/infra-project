FROM nginx:1.27-alpine
COPY index.html /usr/share/nginx/html/
HEALTHCHECK CMD wget -qO- http://localhost/ || exit 1
