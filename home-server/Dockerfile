# --------- BUILD STAGE ---------
FROM node:15-alpine AS build

# Install needed applications
RUN apk add --no-cache \
    git \
    g++ \
    make \
    python

# Clone repository
WORKDIR /usr/src/app
RUN git clone https://github.com/sparcs-kaist/sparcs.org-v2.git .

# Run webpack build
RUN npm install
RUN npm run build

# Delete extraneous devDependencies
RUN npm prune --production

# Move needed files to output directory
RUN mkdir output && \
    mv assets output && \
    mv src output && \
    mv dist output && \
    mv package*.json output && \
    mv node_modules output

# --------- RELEASE STAGE ---------
FROM node:15-alpine AS release

WORKDIR /usr/src/app

# Copy build result and needed files
COPY --from=build /usr/src/app/output .

# Run container
ENV NODE_ENV production
EXPOSE 3000
CMD ["npm", "start"]
