FROM alpine:latest

WORKDIR /app

COPY tabos-backend /app/tabos-backend

RUN chmod +x /app/tabos-backend && \
    apk add --no-cache ca-certificates tzdata && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone

EXPOSE 18080

CMD ["/app/tabos-backend"]
