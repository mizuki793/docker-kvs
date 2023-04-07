From python:3.7.5-slim
ENV COMPOSE_HTTP_TIMEOUT 120
RUN pip install flask==2.2.2 redis==3.3.8
WORKDIR /src
COPY ./src/server.py /src
ENV PORT 80
ENV REDIS_HOST 127.0.0.1
ENV REDIS_PORT 6379
ENV REDIS_DB 0
ENTRYPOINT ["python", "-u", "server.py"]