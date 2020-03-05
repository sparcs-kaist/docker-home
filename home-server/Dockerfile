# --------- BUILD STAGE ---------
FROM node:13-alpine AS build
MAINTAINER SPARCS WHEEL <wheel@sparcs.org>

# Clone repository
WORKDIR /usr/src/app
RUN git clone https://github.com/sparcs-kaist/sparcs.org-v2.git .

# Run webpack build
RUN npm install
RUN npm run build

# Move needed files to output directory
RUN mkdir output && \
    mv assets output && \
    mv src output && \
    mv dist output && \
    mv package*.json output

# --------- RELEASE STAGE ---------
FROM node:13-alpine AS release
MAINTAINER SPARCS WHEEL <wheel@sparcs.org>

WORKDIR /usr/src/app

# Copy build result and needed files
COPY --from=build /usr/src/app/output .

# Install only production dependencies
RUN npm install --only=production
EXPOSE 3000
CMD ["npm", "start"]
