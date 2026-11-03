# Guia de Projetos Práticos: Spring Boot API REST com Microsoft SQL Server (1 Entidade)

**Duração Total:** 8 Horas (4 encontros de 2 horas)  
**Nível:** Ensino Médio / Técnico  
**Foco:** Backend Completo em Java com Spring Boot e Microsoft SQL Server (DDL SQL, Entity, Repository, Controller, Validações e Regras de Negócio)

---

## 📅 Cronograma de Entregas por Aula (4 Semanas / 2h por Encontro)

O cronograma foi planejado para que compreendam todo o fluxo de engenharia de software no backend, iniciando pela **modelagem relacional em SQL Server** (escrita e execução de scripts DDL manuais) até a integração com Spring Data JPA, regras de negócio e testes via cliente HTTP.

```mermaid
gantt
    title Cronograma de Execução: Spring Boot + SQL Server (8 Horas Totais)
    dateFormat  DD/MM
    section Sprint Backend
    Script SQL Server, Conexão e JPA       :done, 09/11, 1d
    Controller REST e Operações Básicas    :active, 16/11, 1d
    Regras de Negócio e Validações:crit, 23/11, 1d
    Tratamento de Exceções & Testes Finais :crit, 30/11, 1d
```

### Aula 1 — 09/11: Script SQL Server, Conexão e Mapeamento JPA
* **Objetivo:** Criar a tabela no Microsoft SQL Server via script DDL, configurar o Spring Boot para conectar ao SQL Server e mapear a `@Entity` correspondente.
* **Entregáveis:**
  * **Script DDL SQL Server (`schema.sql`):** Criação da tabela com tipos nativos do SQL Server (`BIGINT IDENTITY(1,1)`, `VARCHAR`, `DATETIME2`, `BIT`, `DECIMAL`), chaves primárias e restrições `NOT NULL`.
  * Criação do projeto Spring Boot com as dependências: `Spring Web`, `Spring Data JPA`, `MS SQL Server Driver` (`com.microsoft.sqlserver:mssql-jdbc`), `Lombok` e `Validation`.
  * Configuração do `application.properties` com URL de conexão JDBC do SQL Server (`jdbc:sqlserver://...`), usuário, senha e `spring.jpa.hibernate.ddl-auto=validate` (garantindo que o script criado seja respeitado).
  * Classe `@Entity` mapeada com `@Table`, `@Id`, `@GeneratedValue(strategy = GenerationType.IDENTITY)` e anotações `@Column`.
  * Interface `Repository` estendendo `JpaRepository<Entidade, Long>`.
  * Aplicação iniciando com conexão estabelecida e validada com o SQL Server.

### Aula 2 — 16/11: Camada Web e Operações CRUD Básicas
* **Objetivo:** Implementar o `@RestController` e expor as operações fundamentais de leitura, inserção e remoção integradas ao SQL Server.
* **Entregáveis:**
  * Classe `@RestController` com anotação `@RequestMapping("/api/...")`.
  * Endpoint `GET` para listagem completa retornando `ResponseEntity<List<Entidade>>`.
  * Endpoint `GET /{id}` para busca individual com verificação de existência (`Optional` ou `orElseThrow`).
  * Endpoint `POST` para criação inicial do registro (`HttpStatus.CREATED`) persistindo no SQL Server.
  * Endpoint `DELETE /{id}` com retorno `HttpStatus.NO_CONTENT`.
  * Coleção de requisições criada e validada no Postman, Insomnia ou Bruno.

### Aula 3 — 23/11: Regras de Negócio
* **Objetivo:** Implementação da regra central do tema.
* **Entregáveis:**
  * Implementação da **Regra de Negócio** (cálculos automáticos, validações lógicas e transições de status).
  * Endpoint específico exposto no Controller (`PATCH` ou `POST` de ação) invocando a regra de negócio.

### Aula 4 — 30/11: Validações, Tratamento de Erros e Homologação
* **Objetivo:** Implementar validações e finalizar os testes integrados.
* **Entregáveis:**
  * Validações de entrada na requisição (`NotNull`, `NotBlank`, `Min`, `Max`, etc).
  * Retorno de status `400 Bad Request` ou `409 Conflict`.
  * Bateria de testes com cenários de sucesso (status 200/201) e cenários de falha da regra de negócio (status 400/404/409).

---

### 1. Configuração do `application.properties`
```properties
# Conexão com Microsoft SQL Server
spring.datasource.url=jdbc:sqlserver://localhost:1433;databaseName=EscolaDB;encrypt=true;trustServerCertificate=true
spring.datasource.username=sa
spring.datasource.password=SuaSenhaForte123!
spring.datasource.driver-class-name=com.microsoft.sqlserver.jdbc.SQLServerDriver

# Hibernate / JPA
# "validate" valida se o script DDL executado no SQL Server confere exatamente com as anotações Java
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.SQLServerDialect
```

---

## 🏛️ Arquitetura de Camadas Esperada no Projeto

```mermaid
graph TD
    subgraph cliente["Testes de API (Postman / Bruno / Insomnia)"]
        REQ["JSON Request / Endpoints HTTP"]
    end

    subgraph spring["Arquitetura em Camadas (Java)"]
        C["@RestController (Rotas, Regras de Negócio e Códigos HTTP)"]
        R["@Repository (Spring Data JPA)"]
        E["@Entity (Mapeamento Objeto-Relacional)"]
    end

    subgraph banco["Banco de Dados Relacional"]
        SCRIPT["📜 Script DDL (CREATE TABLE)"] -.-> MSSQL[("Microsoft SQL Server")]
        R <-->|"Conexão JDBC (Porta 1433)"| MSSQL
    end

    REQ <-->|HTTP 200, 201, 400, 404| C
    C --> R
    R -.-> E
```

---

## 🚀 O Projeto Prático com Script SQL Server e Regras de Negócio

---

### 8. Playlist e Votação da Rádio do Recreio
* **Contexto:** Os alunos sugerem faixas musicais para tocar no intervalo, com votação pública e aprovação direta ao atingir a meta.
* **Entidade:** `MusicaRecreio`
  * Atributos: `id` (Long), `titulo` (String), `artista` (String), `linkPlataforma` (String), `duracaoSegundos` (Integer), `totalVotos` (Integer), `status` (String: `EM_VOTACAO`, `APROVADA_TOCAR`, `REJEITADA`)
* **Endpoints:**
  * `GET /api/musicas-recreio` — Lista músicas sugeridas para o recreio.
  * `GET /api/musicas-recreio/{id}` — Detalhes da música sugerida.
  * `POST /api/musicas-recreio` — **Regra de Negócio no Cadastro:** Valida se a música tem até 300 segundos (5 minutos) e inicia com `totalVotos = 0` e `status = "EM_VOTACAO"`.
  * `PUT /api/musicas-recreio/{id}` — Edita link ou título antes da aprovação.
  * `DELETE /api/musicas-recreio/{id}` — Remove faixa inadequada.
  * `POST /api/musicas-recreio/{id}/votar` — **Regra de Negócio:** Incrementa voto e aprova automaticamente se atingir 15 votos.
* **Pseudocódigo da Regra de Negócio:**
```text
FUNCAO cadastrarMusica(dados):
    SE dados.duracaoSegundos > 300 ENTAO:
        LANCAR_ERRO("A música não pode exceder o limite de 5 minutos (300 segundos).")
    FIM_SE

    dados.totalVotos = 0
    dados.status = "EM_VOTACAO"

    RETORNAR salvar(dados)
FIM_FUNCAO

FUNCAO votarMusica(id):
    musica = buscarPorId(id)

    SE musica.status != "EM_VOTACAO" ENTAO:
        LANCAR_ERRO("Apenas faixas em votação ativa podem receber votos.")
    FIM_SE

    musica.totalVotos = musica.totalVotos + 1

    SE musica.totalVotos >= 15 ENTAO:
        musica.status = "APROVADA_TOCAR"
    FIM_SE

    RETORNAR salvar(musica)
FIM_FUNCAO
```

---
