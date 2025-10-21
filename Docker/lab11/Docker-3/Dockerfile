
FROM python:3.12-slim

WORKDIR /app

COPY . .

RUN pip install --no-cache-dir flask

ENV APP_MODE=production
ENV APP_REGION=canada-west

EXPOSE 8080

CMD ["python", "app.py"]