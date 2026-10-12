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
RUN yarn build && rm -rf /app/frontend/node_modules

# Stage 2: Build backend
FROM node:lts-alpine AS backend-build

RUN corepack enable && mkdir -p /app/backend

WORKDIR /app

COPY package.json yarn.lock* .yarnrc.yml ./

WORKDIR /app/backend
COPY ./backend/package*.json /app/backend/

RUN yarn workspaces focus --production backend && yarn cache clean

COPY ./backend /app/backend

# Final Stage: Production (最小化)
FROM node:lts-alpine

WORKDIR /app/backend

# 只复制需要的文件
COPY --from=frontend-build /app/frontend/build /app/frontend/build
COPY --from=backend-build /app/backend/package.json /app/backend/package.json
COPY --from=backend-build /app/backend/bin /app/backend/bin
COPY --from=backend-build /app/backend/routes /app/backend/routes
COPY --from=backend-build /app/backend/services /app/backend/services
COPY --from=backend-build /app/backend/utils /app/backend/utils
COPY --from=backend-build /app/backend/app.js /app/backend/app.js
COPY --from=backend-build /app/node_modules /app/backend/node_modules

ENV NODE_ENV=production
ENV ZU_SECURE_HEADERS=true
ENV ZU_SERVE_FRONTEND=true

EXPOSE 4000

CMD ["node", "bin/www.js"]