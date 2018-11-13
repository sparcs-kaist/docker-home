FROM python:3.4-jessie
MAINTAINER SPARCS WHEEL <wheel@sparcs.org>

# Packages installation
RUN apt-get update
RUN apt-get install -y \
    openssh-server \
    sudo \
    vim \
    tmux \
    locales \
    wget \
    cron \
	git \
	mysql-client \
	libmysqlclient-dev \
	gcc \
    apache2 \
    libapache2-mod-wsgi-py3

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

# Certbot installation
RUN wget https://dl.eff.org/certbot-auto
RUN chmod u+x certbot-auto

# Apache configuration
COPY ./home.conf /etc/apache2/sites-available/home.conf
RUN a2enmod ssl
RUN a2enmod proxy_http

# Repository cloning and pip installation
RUN git clone https://github.com/sparcs-kaist/nugu-api.git
WORKDIR /nugu-api
RUN pip install -r requirements.txt

# Crontab installation
RUN echo "# m h dom mon dow command" > mycron
RUN echo "  0 4 *   *   1   /certbot-auto --renew" >> mycron
RUN crontab mycron
RUN rm mycron

CMD /bin/bash -c "service ssh start && service cron start && /bin/bash"
