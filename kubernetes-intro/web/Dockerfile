FROM nginxinc/nginx-unprivileged:1.25-alpine
user 1001:1001
COPY homework.html /app/
COPY default.conf /etc/nginx/conf.d
CMD ["nginx", "-g", "daemon off;"]
