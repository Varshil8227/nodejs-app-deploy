FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --frozen-lockfile && npm cache clean --force

FROM node:20-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ARG VITE_API_URL
ENV VITE_API_URL=$VITE_API_URL
RUN npm run build

FROM nginx:1.25-alpine AS final
RUN addgroup -S webgroup && \
    adduser -S webuser -G webgroup && \
    mkdir -p /tmp/nginx/client_temp \
             /tmp/nginx/proxy_temp \
             /tmp/nginx/fastcgi_temp \
             /var/cache/nginx && \
    chown -R webuser:webgroup \
             /tmp/nginx \
             /var/log/nginx \
             /usr/share/nginx/html \
             /etc/nginx/conf.d \
             /var/cache/nginx
RUN sed -i 's|/var/run/nginx.pid|/tmp/nginx.pid|g' /etc/nginx/nginx.conf && \
    sed -i 's/^user/#user/' /etc/nginx/nginx.conf
COPY --chown=webuser:webgroup nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=builder --chown=webuser:webgroup /app/dist /usr/share/nginx/html
USER webuser
EXPOSE 8080
CMD ["nginx", "-g", "daemon off;"]