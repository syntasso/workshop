FROM "alpine"

LABEL org.opencontainers.image.authors "kratix@syntasso.io"
LABEL org.opencontainers.image.source https://github.com/syntasso/workshop

RUN apk update && apk add --no-cache curl github-cli yq

RUN curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.28.2/bin/linux/amd64/kubectl
RUN chmod u+x kubectl && mv kubectl /bin/kubectl


ADD execute-pipeline execute-pipeline

CMD [ "sh", "-c", "./execute-pipeline" ]
ENTRYPOINT []
