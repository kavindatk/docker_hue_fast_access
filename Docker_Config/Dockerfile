
FROM gethue/hue:4.11.0 


ENV NODE_ENV=production


USER root

RUN pip install --no-cache-dir pyhive presto-python-client sqlalchemy py4j

# Optional: Create a new user if needed
RUN useradd -m -s /bin/bash hadoop && \
    echo "hadoop:hadoop" | chpasswd && \
    usermod -aG sudo hadoop
