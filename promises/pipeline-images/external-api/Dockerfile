FROM "alpine"

LABEL org.opencontainers.image.authors "kratix@syntasso.io"
LABEL org.opencontainers.image.source https://github.com/syntasso/workshop

ARG GH_TOKEN
ENV GH_TOKEN=$GH_TOKEN

RUN apk update && apk add --no-cache curl github-cli yq

ADD execute-pipeline execute-pipeline

CMD [ "sh", "-c", "./execute-pipeline" ]
ENTRYPOINT []
