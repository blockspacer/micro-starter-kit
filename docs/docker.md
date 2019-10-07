# Docker

## Docker Build

```bash
# build
TYPE=srv
TARGET=account
VERSION=0.0.5-SNAPSHOT
# DOCKER_REGISTRY=gcr.io
DOCKER_CONTEXT_PATH=xmlking
# docker build --force-rm=true --rm=true --no-cache \
docker build --rm \
--build-arg VERSION=$VERSION \
--build-arg TYPE=${TYPE} \
--build-arg TARGET=${TARGET} \
--build-arg DOCKER_REGISTRY=${DOCKER_REGISTRY} \
--build-arg DOCKER_CONTEXT_PATH=${DOCKER_CONTEXT_PATH} \
--build-arg VCS_REF=$(git rev-parse --short HEAD) \
--build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
-t ${DOCKER_REGISTRY:+${DOCKER_REGISTRY}/}${DOCKER_CONTEXT_PATH}/${TARGET}-${TYPE}:${VERSION} .

IMANGE_NAME=${DOCKER_REGISTRY:+${DOCKER_REGISTRY}/}${DOCKER_CONTEXT_PATH}/${TARGET}-${TYPE}:${VERSION}

# push
docker push $IMANGE_NAME

# check
docker inspect  $IMANGE_NAME
# remove temp images after build
docker image prune -f
# Remove dangling images
docker rmi $(docker images -f "dangling=true" -q)
# Remove images tagged with vendor=sumo
docker rmi $(docker images -f "label=org.label-schema.vendor=sumo"  -q)
# Remove all <none> layers
docker rmi $(docker images -a|grep "<none>"|awk '$1=="<none>" {print $3}')
```

## Docker Run

> run just for testing image...

```bash
docker run -it \
-e MICRO_SERVER_ADDRESS=0.0.0.0:8080 \
-e MICRO_BROKER_ADDRESS=0.0.0.0:10001 \
-e MICRO_REGISTRY=mdns \
-p 8080:8080 -p 10001:10001 $IMANGE_NAME
```

## Docker Compose Run

Run complete app suite with `docker-compose`

```bash
docker-compose up consul
docker-compose up account-srv
docker-compose up emailer-srv
docker-compose up gateway
docker-compose up account-api
docker-compose up gateway-api
curl "http://localhost:8081/account/AccountService/list?limit=10"
```

## Kubernetes Run

> run just for testing image in k8s...

```bash
# account-srv
kubectl run --rm mytest --image=xmlking/account-srv:latest \
--env="MICRO_REGISTRY=kubernetes" \
--env="MICRO_SELECTOR=static" \
--env="MICRO_SERVER_ADDRESS=0.0.0.0:8080" \
--env="MICRO_BROKER_ADDRESS=0.0.0.0:10001" \
--restart=Never -it

# gateway
kubectl run --rm mygateway --image=microhq/micro:kubernetes \
--env="MICRO_REGISTRY=kubernetes" \
--env="MICRO_SELECTOR=static" \
--restart=Never -it \
--command ./micro api
```