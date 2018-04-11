# dockerfile

FROM node:9.4.0

EXPOSE 8080

RUN apt-get update && apt-get upgrade -y
RUN apt-get install -y \
    build-essential \
    curl \
    sudo \
    wget

ENV APP_HOME=/usr/local/nonroot
ENV APP_LOG=/var/log/app/app.log
ENV APP_ERR=/var/log/app/app.err

# Create a nonroot user and add it as a sudo user
RUN /usr/sbin/useradd --create-home --home-dir $APP_HOME --shell /bin/bash nonroot
RUN /usr/sbin/adduser nonroot sudo
RUN echo "nonroot ALL=NOPASSWD: ALL" >> /etc/sudoers

# Install global npm dependencies
RUN npm install -g \
    nodemon \
    pm2

# Change permissions for folders
RUN mkdir -p /var/log/app && chmod a+w /var/log/app

# Set working directory and switch user
WORKDIR $APP_HOME/app
USER nonroot

# Copy source over
COPY ./ .
RUN sudo chown -R nonroot .

# Install client npm dependencies
WORKDIR $APP_HOME/app/client
RUN npm install

ARG BUILD_MODE=staging

# Build Angular
RUN npm run build:$BUILD_MODE

# Install server npm dependencies
WORKDIR $APP_HOME/app/server
RUN npm install

# Start app
CMD npm start
