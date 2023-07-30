# trustyai-console
Web console for TrustyAI Explainability Service.

## Running the container
Set the address of the TrustyAI service by setting the environment variable `TRUSTYAI_URL` as below:

```shell
 docker run --rm -p 8989:8989 -e TRUSTYAI_URL=http://localhost:8080 trustyai-console
```

The console will then be available at: http://localhost:8989

### Docker compose example
```shell
services:
  console:
    image: trustyai-console
    container_name: console
    ports:
      - "8989:8989"
    environment:
      TRUSTYAI_URL: "http://trustyai:8080"
  trustyai:
    image: trustyai/trustyai-service:999-SNAPSHOT
    container_name: trustyai-service
    ports:
      - "8080:8080"
    environment:
      SERVICE_KSERVE_TARGET: "localhost"
      SERVICE_STORAGE_FORMAT: "MEMORY"
      SERVICE_DATA_FORMAT: "CSV"
      SERVICE_METRICS_SCHEDULE: "5s"
      SERVICE_BATCH_SIZE: 5000
      STORAGE_DATA_FILENAME: "data.csv"
      STORAGE_DATA_FOLDER: "/inputs"
    volumes:
      - ~/volumes/pvc/inputs:/inputs
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    ports:
      - 9090:9090
    restart: unless-stopped
    volumes:
      - ./prometheus:/etc/prometheus
      - prom_data:/prometheus
  generator:
    container_name: generator
    image: trustyai/trustyai-service-generator
    build:
      context: ./logger
      dockerfile: ./generator.Dockerfile
    environment:
      MODEL_NAME: "example-model-1"
      SERVICE_ENDPOINT: "http://trustyai:8080/consumer/kserve/v2/full"
      PYTHONUNBUFFERED: "1"

volumes:
  prom_data:
```

Run `docker compose up -d` and open the console at: http://localhost:8989
