FROM alpine:3.20
LABEL org.opencontainers.image.source="https://github.com/StephanieHhnbrg/kcd-demo"

RUN apk add --no-cache curl unzip && \
    curl -fsSL https://releases.hashicorp.com/http-echo/1.0.0/http-echo_1.0.0_linux_amd64.zip -o http-echo.zip && \
    unzip http-echo.zip && \
    mv http-echo /http-echo && \
    chmod +x /http-echo && \
    rm -rf http-echo.zip

ENV TEXT="💚 Hello from the green deployment!"

ENTRYPOINT [ "sh", "-c" ]
CMD [ "exec /http-echo -text=\"$TEXT\" -listen=:$PORT" ]