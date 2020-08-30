# 6 Containerizing an App

## The Big Picture

APP => Docker File => Docker image build => Container

## Containerizing an App

=> Dockerfile

```dockerfile
# <INSTRUCTION><value>
# FROM = base image
FROM alpine

LABEL maintainer = "asdf@asdf.com"

# Execute Command and create layer on base image
RUN apg add -update nodejs nodejs-npm

#Copy Code in new Layer
COPY . /src

#Add Metadata
WORKDIR /src

# Execute Command and create layer on base image
RUN npm install

EXPOSE 8080

ENTRYPOINT ["node","./app.js"]

```

**Commands**

```bash
docker image build -t imageName .
docker container run -d --name web1 -p 8080:8080 imageName
```

## Digging Deeper

Build Context => Location of Code

**can also point to git repo !**

## Multi Stage Builds

=> Small Packages are better !

```dockerfile
FROM asdf as Stage0


FROM qwerty as Stage1


FROM yxcvb as Stage2
# Copy Code from other stage
COPY --from=Stage0 ...

```