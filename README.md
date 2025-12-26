# Отчёт по практической работе: Дополнительные возможности Docker

## Цель работы

Изучение дополнительных возможностей Docker для управления жизненным циклом контейнеров, мониторинга их состояния и оптимизации использования системных ресурсов.

---

## Подготовка к работе

### Создание рабочей директории и файлов

Создал директорию для практической работы и необходимые файлы:

```bash
mkdir ~/docker-practice
cd ~/docker-practice
```

<img width="479" height="50" alt="Frame 16 (3)" src="https://github.com/user-attachments/assets/1a14e7df-098f-4805-b01b-142fe747faca" />


### Создание файлов entrypoint.sh и dockerfile

Создание скрипта `entrypoint.sh` со следующим содержимым:

```bash
#!/bin/bash

DURATION=${1:-60}

echo "====== Docker Practice Container ======"
echo "Start time: $(date)"
echo "Duration: ${DURATION} seconds"
echo "========================================"

echo "System Information:"
uname -a
echo "Available Memory: $(free -h | grep Mem)"
echo "CPU Info:"
nproc

counter=0
while [ $counter -lt $DURATION ]; do
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] Iteration $((counter + 1))/$DURATION"
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] Memory usage: $(ps aux | awk '{sum+=$6} END {print sum/1024 " MB"}')"
    sleep 5
    counter=$((counter + 5))
done

echo "====== Container Shutdown ======"
echo "End time: $(date)"
echo "Container completed successfully"
```

Создание Dockerfile:

```dockerfile
FROM alpine:3.18

RUN apk add --no-cache \
    bash \
    curl \
    htop \
    procps

RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser

WORKDIR /home/appuser

COPY --chown=appuser:appuser entrypoint.sh /home/appuser/

RUN chmod +x /home/appuser/entrypoint.sh

USER appuser

ENTRYPOINT ["/home/appuser/entrypoint.sh"]

CMD ["60"]
```

### Сборка образа

Выполнил сборку Docker-образа:

```bash
chmod +x entrypoint.sh
docker build -t practice-image:1.0 .
```

<img width="1478" height="603" alt="Frame 15" src="https://github.com/user-attachments/assets/225f8cac-ec84-415a-9968-a6e6d088db16" />



Проверил созданный образ:

```bash
docker images
```

<img width="1030" height="279" alt="Frame 14" src="https://github.com/user-attachments/assets/86b0e180-6757-4eb2-8d2a-81f797a46558" />



---

## Задание 1: Вывод логов контейнера в файл

Запустил контейнер на 30 секунд:

```bash
docker run -d --name practice-container-1 practice-image:1.0 30
```

<img width="1482" height="50" alt="image" src="https://github.com/user-attachments/assets/468b4d84-576c-46bc-8582-bac0ebb336a9" />

Просмотрел логи контейнера в реальном времени:

```bash
docker logs -f practice-container-1
```

<img width="1426" height="550" alt="image" src="https://github.com/user-attachments/assets/f01be881-c1f9-4fb2-96e6-356cdb909fe3" />

Проверил статус завершения контейнера:

```bash
docker ps -a | grep practice-container-1
```

<img width="1420" height="53" alt="Frame 13" src="https://github.com/user-attachments/assets/1fc968e7-5928-48ce-a1cc-993d02f1f44d" />



Сохранил логи в файл:

```bash
docker logs practice-container-1 > ~/docker-practice/container_logs.txt
cat ~/docker-practice/container_logs.txt
```
<img width="1350" height="108" alt="image" src="https://github.com/user-attachments/assets/df3998aa-a3d1-4716-b5ca-3e56f6921d3d" />


и Далее сохранил логи с временными метками и удалил контейнер

---

## Задание 2: Проверка docker-stats


Запустил контейнер на 45 секунд:

```bash
docker run -d --name practice-container-2 practice-image:1.0 45
```

Просмотрел статистику в реальном времени:

```bash
docker stats practice-container-2
```


Сделал снимок статистики:

```bash
docker stats --no-stream practice-container-2
```

<img width="1434" height="50" alt="image" src="https://github.com/user-attachments/assets/07cff6a6-bedd-45e0-b722-06e333bf5d97" />


Сохранил статистику в файл:

```bash
docker stats --no-stream practice-container-2 > ~/docker-practice/stats.txt
cat ~/docker-practice/stats.txt
```

<img width="1486" height="58" alt="Frame 12" src="https://github.com/user-attachments/assets/acdcb36b-7b9f-4d27-aa56-fd966ddd4b51" />



Удалил контейнер после завершения и сделал анализ метрик


---

## Задание 3: Ограничение контейнера по CPU и Memory


Запустил контейнер с ограничениями (256MB памяти, 0.5 CPU):

```bash
docker run -d --name practice-limited \
  --memory=256m \
  --cpus=0.5 \
  practice-image:1.0 60
```

<img width="940" height="125" alt="Frame 11" src="https://github.com/user-attachments/assets/a76b7d41-1261-4043-8187-e335db643826" />



Проверил статистику с учетом лимитов:

```bash
docker stats --no-stream practice-limited
```

Обновил лимиты на работающем контейнере:

```bash
docker update --memory=512m --cpus=1.0 practice-limited
```


Проверил, остановил и удалил контейнер:

```bash
docker stop practice-limited
docker rm practice-limited
```

<img width="1412" height="59" alt="image" src="https://github.com/user-attachments/assets/10c878b8-b8dc-405e-bc92-4eb26cf6f763" />


---

## Задание 4: Сохранение docker-контейнера в tar


Запустил контейнер:

```bash
docker run -d --name practice-export practice-image:1.0 30
```


Проверил статус контейнера:

```bash
docker ps -a | grep practice-export
```

Экспортировал контейнер в tar-архив:

```bash
docker export practice-export > ~/docker-practice/container_export.tar
```

<img width="928" height="25" alt="image" src="https://github.com/user-attachments/assets/c68fbc2e-cf8d-4bc1-9ec8-c5bed46ad03e" />


Проверил размер архива:

```bash
ls -lh ~/docker-practice/container_export.tar
```

Просмотрел содержимое архива:

```bash
tar -tf ~/docker-practice/container_export.tar | head -20
```

<img width="1214" height="520" alt="Frame 9" src="https://github.com/user-attachments/assets/dd510f12-34a8-4432-8481-3f7259217402" />



Создал сжатый архив и удалил контейнер

---

## Задание 5: Загрузка контейнера из tar

Импортировал образ из tar-архива:

```bash
docker import ~/docker-practice/container_export.tar restored-practice:1.0
```

<img width="1385" height="53" alt="Frame 8" src="https://github.com/user-attachments/assets/ec009918-1b54-4540-b35b-2e6c09f8e817" />



Проверил созданный образ и запустил контейнер из восстановленного образа:

```bash
docker run -d --name restored-from-tar restored-practice:1.0 /home/appuser/entrypoint.sh 30
```

<img width="1605" height="51" alt="Frame 7" src="https://github.com/user-attachments/assets/bd129d7c-5707-45d9-90e4-e8ab7a7ea11e" />


Проверил работу контейнера, просмотрел логи восстановленного контейнера:

```bash
sleep 35
docker logs restored-from-tar
```

<img width="1545" height="625" alt="Frame 6" src="https://github.com/user-attachments/assets/ad8a375f-cb23-464f-8033-8010f4056757" />



## Выводы

В ходе практической работы были изучены и успешно применены возможности Docker:
