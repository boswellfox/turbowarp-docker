# Different from gui/ and packager/

FROM node:20-bullseye
WORKDIR /app

COPY src/package*.json ./
RUN npm ci --omit=dev

COPY src/ .
RUN chown -R node:node /app
USER node

ENV PORT=9080
EXPOSE 9080

HEALTHCHECK --interval=30s --timeout=3s \
    CMD ["node", "-e", "require('http').get('http://localhost:9080', r => process.exit(r.statusCode < 500 ? 0 : 1)).on('error', () => process.exit(1))"]

CMD ["npm", "start"]
