FROM node:22-alpine

WORKDIR /app/backend

COPY backend/package*.json ./
RUN npm ci --omit=dev

ENV NODE_PATH=/app/backend/node_modules

COPY backend/ ./
COPY database/ /app/database/
COPY docs/ /app/docs/

ENV NODE_ENV=production
ENV PORT=4001

EXPOSE 4001

CMD ["node", "server.js"]
