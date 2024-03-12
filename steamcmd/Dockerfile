######## BUILDER ########

# Set the base image
FROM docker.io/steamcmd/steamcmd:ubuntu-22 as builder

# Set environment variables
ENV USER root
ENV HOME /root/installer

# Set working directory
WORKDIR $HOME

# Install prerequisites
RUN apt-get update \
    && apt-get install -y --no-install-recommends curl tar

# Download and unpack installer
RUN curl http://media.steampowered.com/installer/steamcmd_linux.tar.gz \
    --output steamcmd.tar.gz --silent
RUN tar -xvzf steamcmd.tar.gz && rm steamcmd.tar.gz

######## INSTALL ########

# Set the base image
FROM docker.io/alpine:3.19

ENV USER="steam"
ENV HOME="/${USER}"
ENV LC_ALL="en_US.UTF-8" \
    LANG="en_US.UTF-8" \
    LANGUAGE="en_US.UTF-8"

# Install prerequisites
RUN apk add --update --no-cache bash && \
    rm -rf /var/cache/apk/*

# Copy steamcmd files from builder
COPY --from=builder /root/installer/steamcmd.sh /usr/lib/games/steam/
COPY --from=builder /root/installer/linux32/steamcmd /usr/lib/games/steam/
COPY --from=builder /usr/games/steamcmd /usr/bin/steamcmd

# Copy required files from builder
COPY --from=builder /etc/ssl/certs /etc/ssl/certs
COPY --from=builder /lib/i386-linux-gnu /lib/
COPY --from=builder /root/installer/linux32/libstdc++.so.6 /lib/

# Add user,group and homedir
RUN adduser -g "${USER}" -h "${HOME}" -D "${USER}"
USER "${USER}"
WORKDIR "${HOME}"

# Update SteamCMD and verify latest version
RUN steamcmd +quit

# Fix steamclient path issue
RUN mkdir -p "${HOME}/.steam" && \
    ln -s "${HOME}/.local/share/Steam/steamcmd/linux32" "${HOME}/.steam/sdk32" && \
    ln -s "${HOME}/.local/share/Steam/steamcmd/linux64" "${HOME}/.steam/sdk64" && \
    ln -s "${HOME}/.steam/sdk32/steamclient.so" "${HOME}/.steam/sdk32/steamservice.so" && \
    ln -s "${HOME}/.steam/sdk64/steamclient.so" "${HOME}/.steam/sdk64/steamservice.so" 

# Set default command
ENTRYPOINT ["steamcmd"]
CMD ["+help", "+quit"]