FROM nginx:alpine

# Copy custom nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copy all static files to nginx html directory
COPY . /usr/share/nginx/html

# Remove backend-java and other non-static files from the container's web root
RUN rm -rf /usr/share/nginx/html/backend-java \
           /usr/share/nginx/html/Dockerfile* \
           /usr/share/nginx/html/docker-compose.yml \
           /usr/share/nginx/html/package.json \
           /usr/share/nginx/html/server.js \
           /usr/share/nginx/html/nginx.conf

EXPOSE 80
