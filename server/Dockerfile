### BASE ###
FROM node:erbium-alpine as base

# Install latest yarn version
RUN yarn global upgrade --silent yarn@latest

### BUILDER STAGE ###
FROM base as builder

# Install dependencies
RUN mkdir /app && chown node:node /app
WORKDIR /app
USER node
COPY --chown=node:node package.json yarn.lock .yarnrc ./
RUN yarn cache clean && yarn install --silent
ENV PATH /app/node_modules/.bin:$PATH

# Copy source code
RUN mkdir /app/server
WORKDIR /app/server
COPY ./ ./

# Build source code
RUN yarn run build

### RUN STAGE ###
FROM base as run

# Expose port & allow Node.js to use priviliged ports
EXPOSE 80
RUN apk add libcap && setcap CAP_NET_BIND_SERVICE=+eip /usr/local/bin/node

# Install dependencies (production only)
RUN mkdir /app && chown node:node /app
WORKDIR /app
USER node
COPY --chown=node:node package.json yarn.lock .yarnrc ./
RUN yarn cache clean && yarn install --production --silent
ENV PATH /app/node_modules/.bin:$PATH

# Copy built source code
RUN mkdir /app/server
WORKDIR /app/server
COPY --chown=node:node --from=builder /app/server/tsconfig*.json ./
COPY --chown=node:node --from=builder /app/server/dist ./

CMD node -r tsconfig-paths/register index.js
