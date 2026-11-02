# ararapay
Um mediador de pagamentos fictício. Apenas um lab para treino de conceitos e tecnologias.

## Stack

- Java 27 + Spring Boot 4.1 (Web MVC, Data JPA, Validation, Flyway, springdoc-openapi)
- PostgreSQL
- Docker / Docker Compose

## Estrutura

```
.
├── compose.yaml      # serviços: postgres (sempre) e api (só no profile "full")
└── api/              # projeto Spring Boot (Maven)
    ├── Dockerfile
    ├── pom.xml
    └── src/main/resources/application.yaml
```

## Configuração

O `application.yaml` não contém dados de conexão com o banco. Eles vêm de variáveis de ambiente,
que o Spring mapeia automaticamente para as propriedades correspondentes (*relaxed binding*):

| Variável de ambiente         | Propriedade                  |
|------------------------------|------------------------------|
| `SPRING_DATASOURCE_URL`      | `spring.datasource.url`      |
| `SPRING_DATASOURCE_USERNAME` | `spring.datasource.username` |
| `SPRING_DATASOURCE_PASSWORD` | `spring.datasource.password` |

As credenciais de desenvolvimento estão definidas no `compose.yaml`:

| Banco         | Usuário         | Senha    |
|---------------|-----------------|----------|
| `ararapay_db` | `ararapay_user` | `secret` |

## Rodando pela IDE (debug)

Pré-requisito: Docker rodando.

### Opção 1: com o suporte a Docker Compose do Spring Boot (padrão)

O projeto usa a dependência `spring-boot-docker-compose`. Ao iniciar a aplicação pela IDE, o Spring Boot:

1. encontra o `compose.yaml`;
2. sobe os serviços que não pertencem a nenhum profile (ou seja, só o `postgres`; a `api` fica de fora por estar no profile `full`);
3. configura o datasource automaticamente a partir do container do postgres.

Nesse modo **não é preciso definir as variáveis `SPRING_DATASOURCE_*`**. Basta rodar/debugar a classe principal da aplicação.

> **Atenção:** com essa feature ligada, as variáveis `SPRING_DATASOURCE_*` são **ignoradas silenciosamente**, mesmo
> que estejam definidas. O Spring Boot lê os dados de conexão do próprio container (a partir dos `POSTGRES_*` do
> `compose.yaml`) e cria um bean de `ConnectionDetails`, que tem precedência sobre as propriedades `spring.datasource.*`.
> Exemplo: uma senha errada em `SPRING_DATASOURCE_PASSWORD` não impede a aplicação de subir.

O Spring procura o `compose.yaml` no *working directory* da aplicação. Como o arquivo está na raiz do repositório
(e não em `api/`), escolha uma das alternativas:

- na run configuration da IDE, defina o working directory como a raiz do repositório; **ou**
- defina a variável de ambiente `SPRING_DOCKER_COMPOSE_FILE=../compose.yaml` (caminho relativo ao working directory `api/`).

### Opção 2: sem o suporte a Docker Compose (banco gerenciado manualmente)

Use esta opção quando quiser que as variáveis `SPRING_DATASOURCE_*` **tenham efeito**, por exemplo para conectar
em um banco diferente do definido no `compose.yaml`. Desligar a feature com `SPRING_DOCKER_COMPOSE_ENABLED=false`
é o que faz o Spring voltar a ler essas variáveis.

Se for usar o postgres do `compose.yaml`, suba o banco primeiro:

```sh
docker compose up -d postgres
```

Depois, defina estas variáveis de ambiente na run configuration da IDE
(IntelliJ: *Run > Edit Configurations... > Environment variables*):

```
SPRING_DOCKER_COMPOSE_ENABLED=false
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/ararapay_db
SPRING_DATASOURCE_USERNAME=ararapay_user
SPRING_DATASOURCE_PASSWORD=secret
```

## Rodando tudo com Docker Compose

Na raiz do repositório:

```sh
docker compose --profile full up --build
```

- `--profile full` inclui o serviço `api` (sem ele, só o `postgres` sobe);
- `--build` reconstrói a imagem da API a partir do `api/Dockerfile`.

Dentro da rede do Compose, a API acessa o banco pelo nome do serviço (`postgres`), e não por `localhost`.
Por isso o `compose.yaml` define `SPRING_DATASOURCE_URL` apontando para `postgres:5432`.

Para parar e remover os containers:

```sh
docker compose --profile full down
```

## Endereços

| O quê      | URL                                   |
|------------|---------------------------------------|
| API        | http://localhost:8080                 |
| Swagger UI | http://localhost:8080/swagger-ui.html |
| PostgreSQL | `localhost:5432`                      |
