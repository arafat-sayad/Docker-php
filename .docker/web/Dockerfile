FROM node:18-alpine

WORKDIR /app

# Install dependencies based on the preferred package manager
COPY ./web .

RUN NODE_ENV=test

RUN yarn install

# Environment variables must be present at build time
# https://github.com/vercel/next.js/discussions/14030
#ARG ENV_VARIABLE
#ENV ENV_VARIABLE=${ENV_VARIABLE}
#ARG NEXT_PUBLIC_ENV_VARIABLE
#ENV NEXT_PUBLIC_ENV_VARIABLE=${NEXT_PUBLIC_ENV_VARIABLE}

# Uncomment the following line to disable telemetry at build time
# ENV NEXT_TELEMETRY_DISABLED 1

RUN NODE_ENV=production

RUN npm i -g pm2

RUN yarn build

CMD pm2-runtime start yarn --interpreter sh --name "web" -- start