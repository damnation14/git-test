# This is a Multi Stage Docker File

# Stage 1
# Use Alpine to reduce the size of docker build
FROM node:12-alpine as builder

# Create app directory
WORKDIR /app
# Install app dependencies
COPY package.json yarn.lock ./
# Downling external dependencies
RUN ["npm", "set", "registry", "https://tknon-registry.tekion.xyz/"]

# RUN npm i -g yarn
RUN ["yarn", "install"]
# Copy everything from directory to current workdir excluding files in dockerignore
COPY . .
RUN ["yarn", "--build", "ENV=local"]
# Remove node_modules here because we will add in stage 2, also we will copy this entire folder in stage 3
RUN ["rm", "-rf", "node_modules"]

# Stage 2
FROM node:12-alpine as dependency
WORKDIR /app
# Download production dependency alone

COPY ./packages/tap-builder/package.json ./packages/tap-builder/package.json
COPY ./packages/tap-user/package.json ./packages/tap-user/package.json

COPY ./packages/tap-builder/server/package.json ./packages/tap-builder/server/package.json
COPY ./packages/tap-user/server/package.json ./packages/tap-user/server/package.json

RUN ["npm", "set", "registry", "https://tknon-registry.tekion.xyz/"]
RUN yarn install --production=true
# couchbase sdk requirements
#RUN ["apk", "update", "&&", "apk", "add", "curl", "bash", "&&", "rm", "-rf", "/var/cache/apk/*"]
# install node-prune (https://github.com/tj/node-prune)
COPY node-prune ./
RUN apk update
RUN apk add curl

RUN curl -sf https://gobinaries.com/tj/node-prune | sh
# node-prune removes unncessary files in node_modules

ENTRYPOINT ["/node-prune.sh"]


# Stage 3
FROM node:12-alpine
WORKDIR /app
COPY --from=builder /app/. ./
COPY --from=dependency /app/node_modules ./node_modules
EXPOSE 4112
WORKDIR ./packages/tap-builder/
CMD ["yarn", "start"]