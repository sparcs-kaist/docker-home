FROM node:8-jessie
MAINTAINER SPARCS WHEEL <wheel@sparcs.org>

# Packages installation
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
RUN echo "deb http://repo.mongodb.org/apt/debian jessie/mongodb-org/3.4 main" | tee /etc/apt/sources.list.d/mongodb-org-3.4.list
RUN apt-get update
RUN apt-get install -y \
    openssh-server \
    sudo \
    vim \
    tmux \
    locales \
    wget \
	git \
    mongodb-clients \
    stunnel4 \
    nfs-common \
    binutils
RUN git clone https://github.com/aws/efs-utils
WORKDIR /efs-utils
RUN ./build-deb.sh
RUN dpkg -i ./build/amazon-efs-utils*deb

# Creating users
RUN adduser --gecos "" --disabled-password sysop
RUN usermod -G sudo sysop
RUN adduser --gecos "" --disabled-password wheel
RUN usermod -G sudo wheel

# Locale configuration
RUN sed -i -e 's/# ko_KR.UTF-8 UTF-8/ko_KR.UTF-8 UTF-8/' /etc/locale.gen
RUN dpkg-reconfigure --frontend=noninteractive locales
RUN update-locale LANG=ko_KR.UTF-8
RUN echo "export LANG=ko_KR.UTF-8" >> /etc/environment
RUN echo "export LANGUAGE=ko_KR.UTF-8" >> /etc/environment
RUN echo "export LC_ALL=ko_KR.UTF-8" >> /etc/environment

# Repository cloning and gulp installation
RUN git clone https://github.com/sparcs-kaist/sparcs.org.git
COPY localconfig.js /sparcs.org/localconfig.js
COPY semantic /sparcs.org/semantic
COPY semantic.json /sparcs.org/semantic.json
WORKDIR /sparcs.org
RUN npm install -g gulp

# Semantic UI installation
WORKDIR /sparcs.org/semantic
RUN npm install gulp
RUN gulp build

# Node modules installation
WORKDIR /sparcs.org
RUN grep -v semantic-ui package.json > package.json.new
RUN mv package.json.new package.json
RUN npm install
RUN npm run build
RUN npm install -g forever

CMD /bin/bash -c "service ssh start && NODE_ENV=production PORT=15693 forever start --id home /sparcs.org/build/server.js && /bin/bash "
