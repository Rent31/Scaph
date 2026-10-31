FROM python:3.11-slim

WORKDIR /scaph/web

COPY scaph-api .
COPY shared ./shared
COPY console ./console
COPY billing ./billing

EXPOSE 8080

CMD ["python3", "scaph-api"]
