FROM golang:1.18-alpine as builder

WORKDIR /app

COPY . ./

RUN go mod download
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 GO111MODULE=on go build -a -o todo-server main.go

# Use distroless as minimal base image to package the manager binary
# Refer to https://github.com/GoogleContainerTools/distroless for more details
FROM gcr.io/distroless/static:nonroot
WORKDIR /app
COPY --from=builder /app/todo-server .
COPY --from=builder /app/public/ ./public/
COPY --from=builder /app/views/ ./views/

EXPOSE 8080

CMD [ "/app/todo-server" ]