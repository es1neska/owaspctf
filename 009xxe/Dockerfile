FROM ubuntu:20.04

LABEL maintainer="Gabe Marshall <gabemarshall@me.com>"

# Update and install dependencies
RUN apt-get -yqq update && \
    apt-get -yqq install --allow-unauthenticated openjdk-8-jdk maven gopher && \
    rm -rf /var/lib/apt/lists/*

# Copy project files and add FLAG to /etc/passwd
COPY ./ ./
RUN echo 'FLAG{Xuman_Like_Machines}' >> /etc/passwd

# Start the application
CMD ["mvn", "jetty:run"]