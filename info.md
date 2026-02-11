# 📝 Заметки о Docker

## ⚡ Быстрые команды и шпаргалки

### Базовые команды Docker
```bash
# Управление образами
docker pull <image>           # Скачать образ
docker push <image>           # Загрузить образ
docker build -t <name> .     # Собрать образ
docker images                # Список образов
docker rmi <image>           # Удалить образ
docker tag <image> <tag>     # Присвоить тег

# Управление контейнерами
docker run <image>           # Запустить контейнер
docker run -d -p 80:80 nginx # Запустить в фоне с пробросом порта
docker ps                    # Запущенные контейнеры
docker ps -a                 # Все контейнеры
docker stop <container>      # Остановить
docker start <container>     # Запустить остановленный
docker restart <container>   # Перезапустить
docker rm <container>        # Удалить контейнер
docker logs <container>      # Логи контейнера
docker exec -it <container> bash  # Войти в контейнер

# Очистка системы
docker system prune          # Очистить неиспользуемые ресурсы
docker system prune -a       # Полная очистка (включая остановленные контейнеры)
docker volume prune          # Удалить неиспользуемые тома
docker network prune         # Удалить неиспользуемые сети
```

### Docker Compose
```bash
# Основные команды
docker-compose up -d        # Запустить в фоне
docker-compose down         # Остановить и удалить контейнеры
docker-compose down -v     # Удалить также тома
docker-compose logs -f     # Логи всех сервисов
docker-compose logs -f web # Логи конкретного сервиса
docker-compose ps          # Статус сервисов
docker-compose exec web sh # Выполнить команду в сервисе
docker-compose restart     # Перезапустить все сервисы
docker-compose build       # Пересобрать образы
docker-compose pull        # Обновить образы
```

## 🐳 Полезные Dockerfile инструкции

### Часто используемые комбинации

```dockerfile
# Минимальный Node.js образ
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
USER node
CMD ["node", "index.js"]

# Python с зависимостями
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "app.py"]

# Многостадийная сборка Go
FROM golang:1.21 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN go build -o main .

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
CMD ["./main"]
```

## 🔍 Диагностика и отладка

### Проверка контейнера
```bash
# Информация о контейнере
docker inspect <container>
docker inspect --format='{{.NetworkSettings.IPAddress}}' <container>

# Ресурсы
docker stats                 # Использование ресурсов
docker top <container>       # Процессы в контейнере

# Копирование файлов
docker cp <container>:/app/logs ./logs/
docker cp ./config.json <container>:/app/config.json
```

### Частые проблемы и решения

**Проблема**: Контейнер сразу завершается
```bash
# Решение: Проверить логи и запустить интерактивно
docker logs <container>
docker run -it <image> /bin/sh
```

**Проблема**: Нет доступа к сети
```bash
# Решение: Проверить сетевые настройки
docker network ls
docker network inspect bridge
docker run --network host <image>
```

**Проблема**: Закончилось место
```bash
# Решение: Очистка
docker system prune -a --volumes
docker rmi $(docker images -f "dangling=true" -q)
```

## 📦 Работа с Docker Hub

### Аутентификация и публикация
```bash
# Вход в Docker Hub
docker login
docker login -u username -p password

# Тегирование и публикация
docker tag myapp:latest username/myapp:1.0.0
docker push username/myapp:1.0.0
docker push username/myapp:latest

# Поиск образов
docker search nginx --limit 10
docker search --filter "is-official=true" python
```

### Полезные официальные образы
```bash
# Легковесные базовые образы
alpine:3.19          # ~5MB
busybox:latest       # ~1.5MB
debian:bullseye-slim # ~80MB

# Специализированные образы
node:18-alpine       # Node.js
python:3.12-slim     # Python
golang:1.21-alpine   # Go
nginx:alpine         # Nginx
postgres:16-alpine   # PostgreSQL
redis:7-alpine       # Redis
```

## 🛠️ Продвинутые техники

### Кэширование в Docker
```dockerfile
# Порядок инструкций влияет на кэширование
# 1. Редко изменяемые зависимости
COPY package.json package-lock.json ./
RUN npm ci

# 2. Часто изменяемый код
COPY . .
RUN npm run build
```

### Healthcheck
```dockerfile
# В Dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# В docker-compose.yml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
  interval: 30s
  timeout: 3s
  retries: 3
  start_period: 5s
```

### Метки и аннотации
```dockerfile
# Полезные метки для организации
LABEL org.opencontainers.image.title="My App"
LABEL org.opencontainers.image.description="Description here"
LABEL org.opencontainers.image.version="1.0.0"
LABEL org.opencontainers.image.created="2024-01-01"
LABEL org.opencontainers.image.authors="dev@example.com"
LABEL org.opencontainers.image.licenses="MIT"
```

## 💾 Работа с данными

### Тома (Volumes)
```bash
# Создание и использование томов
docker volume create app-data
docker volume ls
docker volume inspect app-data

# Подключение тома
docker run -v app-data:/data myapp
docker run --mount source=app-data,target=/data myapp

# Бэкап тома
docker run --rm -v app-data:/source -v $(pwd):/backup alpine \
  tar czf /backup/app-data-backup.tar.gz -C /source .
```

### Bind mounts
```bash
# Для разработки
docker run -v $(pwd):/app -v /app/node_modules myapp
docker run --mount type=bind,source="$(pwd)",target=/app myapp
```

## 🔐 Безопасность

### Пользователи и права
```dockerfile
# Создание непривилегированного пользователя
RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup
USER appuser

# Или использование существующего
USER node
```

### Сканирование уязвимостей
```bash
# Docker Scout
docker scout quickview nginx:latest
docker scout cves nginx:latest
docker scout compare nginx:latest nginx:1.25

# Trivy (сторонний инструмент)
trivy image nginx:latest
```

## 🚀 Оптимизация

### Уменьшение размера образа
```dockerfile
# Использование --no-cache и очистка в одной инструкции
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        package1 \
        package2 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Объединение RUN инструкций
RUN command1 && \
    command2 && \
    command3

# Использование .dockerignore
.git
node_modules
*.md
Dockerfile
docker-compose.yml
.env
```

### Производительность
```bash
# Лимиты ресурсов
docker run --memory="256m" --cpus="0.5" nginx

# Режим restart
docker run --restart=unless-stopped nginx
docker run --restart=on-failure:5 nginx
```

## 📝 Шаблоны docker-compose.yml

### Разработка (горячая перезагрузка)
```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DEBUG=true
    command: npm run dev
```

### Production-ready
```yaml
version: '3.8'
services:
  web:
    image: registry.example.com/myapp:${TAG:-latest}
    restart: unless-stopped
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

### Локальный стек с базами данных
```yaml
version: '3.8'
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypass
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
  
  redis:
    image: redis:7-alpine
    command: redis-server --requirepass redispass
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

## 🎯 Docker в CI/CD

### GitHub Actions
```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: username/myapp:latest
```

### GitLab CI
```yaml
docker-build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA .
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
```

## 🔧 Полезные алиасы

Добавьте в `~/.bashrc` или `~/.zshrc`:

```bash
# Docker алиасы
alias d='docker'
alias dc='docker-compose'
alias dps='docker ps'
alias dpsa='docker ps -a'
alias di='docker images'
alias drmi='docker rmi'
alias drm='docker rm'
alias dst='docker stop'
alias dsta='docker stop $(docker ps -q)'
alias dcl='docker logs'
alias dcf='docker-compose logs -f'
alias dcup='docker-compose up -d'
alias dcdown='docker-compose down'
alias dcrestart='docker-compose restart'
alias dcb='docker-compose build'
alias dce='docker-compose exec'
alias dclogs='docker-compose logs'

# Очистка
alias dprune='docker system prune -a --volumes'
alias dprunei='docker rmi $(docker images -f "dangling=true" -q)'
alias dprunec='docker rm $(docker ps -a -q)'

# Мониторинг
alias dtop='docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"'
```

## 📊 Мониторинг и логирование

### Просмотр логов
```bash
# Фильтрация логов
docker logs --tail 50 <container>
docker logs --since 1h <container>
docker logs --until 30m <container>

# Форматированный вывод
docker inspect --format='{{.Name}} - {{.State.Status}}' $(docker ps -aq)
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"
```

### Мониторинг ресурсов
```bash
# Статистика контейнеров
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemPerc}}"

# Поиск контейнеров по использованию памяти
docker stats --no-stream | sort -k 4 -h
```

## 💡 Советы и рекомендации

1. **Всегда тегируйте образы** - не используйте `latest` в production
2. **Используйте .dockerignore** - как .gitignore, но для Docker
3. **Один процесс на контейнер** - разделяйте приложения по разным контейнерам
4. **Минимизируйте слои** - объединяйте команды RUN
5. **Не храните секреты в образах** - используйте переменные окружения
6. **Используйте read-only rootfs** - `docker run --read-only`
7. **Проверяйте здоровье контейнеров** - используйте HEALTHCHECK
8. **Регулярно обновляйте базовые образы** - для безопасности
9. **Используйте специфичные теги** - не `node:latest`, а `node:18.17.0-alpine`
10. **Документируйте порты** - явно указывайте EXPOSE

---

*Этот файл содержит практические заметки и будет дополняться. 
Последнее обновление: 2026 год.*