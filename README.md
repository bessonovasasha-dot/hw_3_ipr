## дз ипр
Условие дз доступно по [ссылке](https://github.com/Nikfilk2030/missing_semester_hse_cs/blob/main/2026/amcs/year_1/hw_3/README.md)

текст докерфайл:

FROM python:3.13-slim-bookworm

WORKDIR /app

COPY app/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app/app.py .
RUN mkdir /data && chown 10001:10001 /data
USER 10001:10001
EXPOSE 8000
CMD ["python", "app.py"]

выводы счетчика и логи:

sashabessonova@192 hw_3_ipr % curl http://127.0.0.1:8080
{"greeting": "Привет, Docker!", "count": 0, "hostname": "348a51f5ddb6", "storage": "sqlite"}%                                                                   
sashabessonova@192 hw_3_ipr % curl -X POST http://127.0.0.1:8080/increment
{"greeting": "Привет, Docker!", "count": 1, "hostname": "348a51f5ddb6", "storage": "sqlite"}%                                                                   
sashabessonova@192 hw_3_ipr % curl http://127.0.0.1:8080
{"greeting": "Привет, Docker!", "count": 1, "hostname": "348a51f5ddb6", "storage": "sqlite"}%                                                                   
sashabessonova@192 hw_3_ipr % docker logs ipr-counter
2026-11-24 16:38:51,183 listening port=8000 storage=sqlite
2026-11-24 16:39:22,221 http 172.17.0.1 "GET / HTTP/1.1" 200 -
2026-11-24 16:39:36,932 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-24 16:39:55,027 http 172.17.0.1 "GET / HTTP/1.1" 200 -


файл .env.example:

APP_PORT=8080
GREETING=Привет из Compose

файл compose.yaml:

services:
  app:
    build: .
    ports:
      - "${APP_PORT:-8080}:8000"
    environment:
      - GREETING=${GREETING:-Hello}
      - DATA_DIR=/data
    volumes:
      - ipr-notes:/data

volumes:
  ipr-notes:

проверка сохранения счетчика:

sashabessonova@192 hw_3_ipr % docker compose down
docker compose up -d
docker compose ps
[+] down 2/2
 ✔ Container hw_3_ipr-app-1 Removed                                         3.2s
 ✔ Network hw_3_ipr_default Removed                                         0.1s
[+] up 2/2
 ✔ Network hw_3_ipr_default Created                                         0.0s
 ✔ Container hw_3_ipr-app-1 Started                                         0.1s
NAME             IMAGE          COMMAND           SERVICE   CREATED        STATUS                  PORTS
hw_3_ipr-app-1   hw_3_ipr-app   "python app.py"   app       1 second ago   Up Less than a second   0.0.0.0:8080->8000/tcp, [::]:8080->8000/tcp
sashabessonova@192 hw_3_ipr % curl http://127.0.0.1:8080
{"greeting": "hello from compose:)", "count": 1, "hostname": "7400bb497ffd", "storage": "sqlite"}%                                                              
sashabessonova@192 hw_3_ipr % curl -X POST http://127.0.0.1:8080/increment
curl http://127.0.0.1:8080
{"greeting": "hello from compose:)", "count": 2, "hostname": "7400bb497ffd", "storage": "sqlite"}{"greeting": "hello from compose:)", "count": 2, "hostname": "7400bb497ffd", "storage": "sqlite"}%                                             
sashabessonova@192 hw_3_ipr % docker compose down
[+] down 2/2
 ✔ Container hw_3_ipr-app-1 Removed                                         3.2s
 ✔ Network hw_3_ipr_default Removed                                         0.1s
sashabessonova@192 hw_3_ipr % docker compose up -d
[+] up 2/2
 ✔ Network hw_3_ipr_default Created                                         0.0s
 ✔ Container hw_3_ipr-app-1 Started                                         0.1s
sashabessonova@192 hw_3_ipr % curl http://127.0.0.1:8080
{"greeting": "hello from compose:)", "count": 2, "hostname": "83dc98815418", "storage": "sqlite"}% 



файл compose.yaml

services:
  app:
    build: .
    ports:
      - "${APP_PORT:-8080}:8000"
    environment:
      - GREETING=${GREETING:-Hello}
      - STORAGE=redis
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "python", "-c", "import urllib.request; urllib.request.urlopen('http://localhost:8000/healthz')"]
      interval: 5s
      timeout: 3s
      retries: 3

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 3

volumes:
  redis-data:

sashabessonova@192 hw_3_ipr % docker compose down
docker compose up -d
curl http://127.0.0.1:8080
[+] down 3/3
 ✔ Container hw_3_ipr-app-1   Removed                                       3.2s
 ✔ Container hw_3_ipr-redis-1 Removed                                       0.1s
 ✔ Network hw_3_ipr_default   Removed                                       0.1s
[+] up 3/3
 ✔ Network hw_3_ipr_default   Created                                       0.0s
 ✔ Container hw_3_ipr-redis-1 Healthy                                       5.6s
 ✔ Container hw_3_ipr-app-1   Started                                       5.7s
curl: (56) Recv failure: Connection reset by peer

sashabessonova@192 hw_3_ipr % curl http://127.0.0.1:8080
{"greeting": "hello from compose:)", "count": 2, "hostname": "e7cff8e325e3", "storage": "redis"}%                                                               
sashabessonova@192 hw_3_ipr % docker compose stop redis
[+] stop 1/1
 ✔ Container hw_3_ipr-redis-1 Stopped                                       0.2s
sashabessonova@192 hw_3_ipr % curl -i http://127.0.0.1:8080/healthz
HTTP/1.0 200 OK
Server: BaseHTTP/0.6 Python/3.13.18
Date: Tue, 24 Nov 2026 17:32:20 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 19
Cache-Control: no-store

{"status": "alive"}%                                                            sashabessonova@192 hw_3_ipr % curl -i http://127.0.0.1:8080/readyz
HTTP/1.0 503 Service Unavailable
Server: BaseHTTP/0.6 Python/3.13.18
Date: Tue, 24 Nov 2026 17:32:48 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 32
Cache-Control: no-store

{"error": "storage_unavailable"}%                                               sashabessonova@192 hw_3_ipr % curl -i -X POST http://127.0.0.1:8080/increment
HTTP/1.0 503 Service Unavailable
Server: BaseHTTP/0.6 Python/3.13.18
Date: Tue, 24 Nov 2026 17:32:58 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 32
Cache-Control: no-store

{"error": "storage_unavailable"}%                                               sashabessonova@192 hw_3_ipr % docker compose start redis
[+] start 1/1
 ✔ Container hw_3_ipr-redis-1 Started                                       0.1s
sashabessonova@192 hw_3_ipr % docker compose ps
curl -i http://127.0.0.1:8080/readyz
curl http://127.0.0.1:8080
curl -X POST http://127.0.0.1:8080/increment
curl http://127.0.0.1:8080
NAME               IMAGE            COMMAND                  SERVICE   CREATED         STATUS                    PORTS
hw_3_ipr-app-1     hw_3_ipr-app     "python app.py"          app       2 minutes ago   Up 2 minutes (healthy)    0.0.0.0:8080->8000/tcp, [::]:8080->8000/tcp
hw_3_ipr-redis-1   redis:7-alpine   "docker-entrypoint.s…"   redis     2 minutes ago   Up 24 seconds (healthy)   6379/tcp
HTTP/1.0 200 OK
Server: BaseHTTP/0.6 Python/3.13.18
Date: Tue, 24 Nov 2026 17:33:31 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 39
Cache-Control: no-store

{"status": "ready", "storage": "redis"}{"greeting": "hello from compose:)", "count": 2, "hostname": "e7cff8e325e3", "storage": "redis"}{"greeting": "hello from compose:)", "count": 3, "hostname": "e7cff8e325e3", "storage": "redis"}{"greeting": "hello from compose:)", "count": 3, "hostname": "e7cff8e325e3", "storage": "redis"}%       