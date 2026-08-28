FROM node:20-bullseye AS build
WORKDIR /app

COPY src/ .
RUN npm ci && npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
# scratch-gui used build/, packager uses dist/
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
HEALTHCHECK --interval=30s --timeout=3s \
    CMD ["sh", "-c", "wget -qO- http://localhost/ || exit 1"]
