FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

# Install necessary dependencies and clean up
RUN apt-get update && apt-get install -y \
    wget \
    curl \
    gnupg \
    lsb-release \
    && rm -rf /var/lib/apt/lists/*

# Add Perforce repository and install P4D
RUN curl -fsSL https://package.perforce.com/perforce.pubkey | gpg --dearmor -o /usr/share/keyrings/perforce-archive-keyring.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/perforce-archive-keyring.gpg] https://package.perforce.com/apt/ubuntu $(lsb_release -cs) release" | tee /etc/apt/sources.list.d/perforce.list \
    && apt-get update \
    && apt-get install -y helix-p4d \
    && rm -rf /var/lib/apt/lists/*

# Create a directory for Perforce data
RUN mkdir -p /opt/perforce

# Set environment variables
ENV P4ROOT /opt/perforce
ENV P4PORT 1666

# Expose the default Perforce port
EXPOSE 1666

# Create a verbose startup script with additional debugging information
RUN echo '#!/bin/bash\n\
echo "Debugging information:"\n\
echo "Contents of /opt/perforce:"\n\
ls -la /opt/perforce\n\
echo "Checking helix-p4d package:"\n\
dpkg -l | grep helix-p4d\n\
echo "Locating p4d:"\n\
P4D_PATH=$(dpkg -L helix-p4d 2>/dev/null | grep -E "/p4d$" | head -n 1)\n\
echo "p4d path: $P4D_PATH"\n\
if [ -n "$P4D_PATH" ]; then\n\
    if [ -x "$P4D_PATH" ]; then\n\
        echo "p4d is executable"\n\
        echo "Starting p4d..."\n\
        exec "$P4D_PATH" -p 1666 -r /opt/perforce -L /var/log/p4d.log\n\
    else\n\
        echo "Error: p4d found but not executable"\n\
        ls -l "$P4D_PATH"\n\
    fi\n\
else\n\
    echo "Error: p4d not found"\n\
    echo "Searching for p4d in common locations:"\n\
    find / -name p4d 2>/dev/null\n\
    echo "Listing all files from helix-p4d package:"\n\
    dpkg -L helix-p4d\n\
fi\n\
echo "System information:"\n\
uname -a\n\
lsb_release -a\n\
echo "Installed packages:"\n\
dpkg -l | grep perforce\n\
exit 1' > /start.sh \
&& chmod +x /start.sh

# Use the startup script as the entrypoint
CMD ["/start.sh"]