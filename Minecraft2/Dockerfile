# syntax=docker/dockerfile:1

# Etapa 1: compilar el .jar con Maven (las pruebas se ejecutan aparte, en CI)
FROM maven:3.9.16-eclipse-temurin-17-noble AS build
WORKDIR /src
COPY pom.xml .
RUN --mount=type=cache,target=/root/.m2 mvn -B -ntp -q dependency:go-offline
# El código y los recursos (texturas en src/main/resources) están bajo src/
COPY src ./src
# El plugin shade deja dos .jar en target/; se copia el ejecutable por su nombre exacto
RUN --mount=type=cache,target=/root/.m2 mvn -B -ntp -q package -DskipTests \
 && cp target/minecraft2-0.1.0-SNAPSHOT.jar /app.jar

# Etapa 2: imagen final solo con el JRE y el .jar
FROM eclipse-temurin:17.0.20.1_1-jre-noble
# Usuario sin privilegios y carpeta de mundos con sus permisos (se monta como volumen)
RUN groupadd --system app && useradd --system --gid app --no-create-home app \
 && mkdir /data && chown app:app /data
WORKDIR /app
COPY --from=build --chown=app:app /app.jar app.jar
USER app

# Modo consola (no hay pantalla ni GPU en un contenedor) y mundos en /data/worlds
ENTRYPOINT ["java", "-XX:MaxRAMPercentage=75", "-Dfile.encoding=UTF-8", "-Dmc2.console=true", "-Dmc2.worlds.dir=/data/worlds", "-jar", "/app/app.jar"]
