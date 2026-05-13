FROM node:25-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci --include=dev

COPY . .
RUN npx prisma generate && npm run build

FROM node:25-alpine

WORKDIR /app

COPY --from=builder /app/package*.json ./
RUN npm ci --only=production

COPY --from=builder /app/dist/src ./dist
COPY --from=builder /app/src/generated ./src/generated
COPY --from=builder /app/prisma ./prisma
COPY --from=builder /app/prisma.config.ts ./prisma.config.ts

EXPOSE 3000

CMD ["node", "dist/main"]
