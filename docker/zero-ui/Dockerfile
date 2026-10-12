# Stage 1: Build frontend
FROM --platform=$BUILDPLATFORM node:lts-alpine AS frontend-build

ENV GENERATE_SOURCEMAP=false

RUN corepack enable && mkdir -p /app/frontend

WORKDIR /app

COPY tsconfig.json package.json yarn.lock* .yarnrc.yml ./

WORKDIR /app/frontend
COPY ./frontend/package*.json /app/frontend/

RUN yarn workspaces focus frontend
COPY ./frontend /app/frontend/
RUN yarn build

# Stage 2: Build backend
FROM node:lts-alpine AS backend-build

RUN corepack enable && mkdir -p /app/backend

WORKDIR /app

COPY package.json yarn.lock* .yarnrc.yml ./

WORKDIR /app/backend
COPY ./backend/package*.json /app/backend/

RUN yarn workspaces focus --production backend && yarn cache clean

COPY ./backend /app/backend

# Final Stage: Production
FROM node:lts-alpine

WORKDIR /app/backend

COPY --from=frontend-build /app/frontend/build /app/frontend/build
COPY --from=backend-build /app/backend /app/backend
COPY --from=backend-build /app/node_modules /app/backend/node_modules

ENV NODE_ENV=production
ENV ZU_SECURE_HEADERS=true
ENV ZU_SERVE_FRONTEND=true

EXPOSE 4000

CMD ["node", "bin/www.js"]