FROM ubuntu
MAINTAINER viki-data data@viki.com

RUN echo "deb http://archive.ubuntu.com/ubuntu precise universe" >> /etc/apt/sources.list
RUN apt-get update
RUN apt-get install -y unzip openjdk-6-jdk wget supervisor

RUN wget -q -N https://dl.dropboxusercontent.com/s/dj86w8ojecgsam7/storm-0.9.0.1.zip
RUN unzip -o /storm-0.9.0.1.zip -d /usr/share/
RUN rm storm-0.9.0.1.zip

ENV STORM_HOME /usr/share/storm-0.9.0.1

RUN groupadd storm
RUN useradd --gid storm --home-dir /home/storm --create-home --shell /bin/bash storm
RUN chown -R storm:storm $STORM_HOME
RUN mkdir /var/log/storm
RUN chown -R storm:storm /var/log/storm
RUN ln -s $STORM_HOME/bin/storm /usr/bin/storm

# Add /mnt/storm directory (used for storing storm settings)
RUN mkdir -v /mnt/storm
RUN chown -R storm:storm /mnt/storm

ADD storm.yaml $STORM_HOME/conf/storm.yaml
ADD cluster.xml $STORM_HOME/logback/cluster.xml
ADD config-supervisord.sh /usr/bin/config-supervisord.sh
ADD start-supervisor.sh /usr/bin/start-supervisor.sh

RUN echo [supervisord] | tee -a /etc/supervisor/supervisord.conf
RUN echo nodaemon=true | tee -a /etc/supervisor/supervisord.conf