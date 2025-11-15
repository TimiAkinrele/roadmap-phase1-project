FROM nginx:1.25-alpine

COPY nginx/nginx.conf /etc/nginx/nginx.conf

# Copy our static HTML file from the web folder to Nginx's web root
COPY web/static/index.html /usr/share/nginx/html/index.html
