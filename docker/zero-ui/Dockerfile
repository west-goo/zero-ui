# Stage 1: Build frontend
FROM --platform=$BUILDPLATFORM node:lts-alpine AS frontend-build
ENV GENERATE_SOURCEMAP=false
RUN corepack enable && mkdir -p /app/frontend
WORKDIR /app
COPY tsconfig.json package.json yarn.lock* .yarnrc.yml ./
WORKDIR /app/frontend
COPY ./frontend/package*.json ./
RUN yarn workspaces focus frontend
COPY ./frontend .
RUN yarn build && rm -rf node_modules public/locales/en public/locales/es-ES public/locales/ru-RU

# Stage 2: Build backend
FROM node:lts-alpine AS backend-build
RUN corepack enable && mkdir -p /app/backend
WORKDIR /app
COPY package.json yarn.lock* .yarnrc.yml ./
WORKDIR /app/backend
COPY ./backend/package*.json ./
RUN yarn workspaces focus --production backend && yarn cache clean
COPY ./backend .
RUN find . -name "*.md" -delete -o -name "*.test.js" -delete -o -name "*.map" -delete

# Final Stage: Production
FROM node:lts-alpine
WORKDIR /app/backend
COPY --from=frontend-build /app/frontend/build /app/frontend/build
COPY --from=backend-build /app/backend .
COPY --from=backend-build /app/node_modules ./node_modules
RUN find node_modules -name "*.md" -delete -o -name "*.map" -delete -o -name "*.test.js" -delete
ENV NODE_ENV=production ZU_SECURE_HEADERS=true ZU_SERVE_FRONTEND=true
EXPOSE 4000
CMD ["node", "bin/www.js"]