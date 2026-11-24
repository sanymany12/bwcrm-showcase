# BW CRM

It's also my main learning project. I started it being experienced in Java but not how production Spring applications are usually structured,
and I use it to learn that on a real problem instead of a tutorial. I wanted to stick to Java and focus mainly on the backend, so I choose the Vaadin architecture wich allows for a faster workflow.

> [!NOTE]
> The source code is private because this is a real system used by our business. This repo documents what I built and why. I'm happy to walk through the code live over screen share in an interview.

**Status:** in active development, currently at milestone [M_2] of 9 (a public demo with sample data is planned for M9).

## What's in it
 
- Data binders, and server side front end components with Vaadin
- Authentication and role-based access with Spring Security
- Versioned database migrations, central error handling
- Docker Compose dev environment, CI that runs the tests on every push
## How I use AI on this project
 
I build this project with Claude Code, and I use it more like a senior pair programmer than an autocomplete. Since I didn't know the common industry patterns yet, most architectural choices (layering, migrations, security setup) came from its suggestions.
 
My part:
 
- **Requirements and priorities:** The overall plan. What the system needs to do, divided into milestones.
- **Understanding before accepting:** The plan is one of the most important steps, and the beginning of the workflow. I don't commit code I can't explain. I always make sure to understand the concepts
- **Review and testing:** Review and run the tests, and do a thourugh manual testing. Push and merge only with a 100% certanty
- **Pushing back:** In some cases, even after code review by me and a subagent, there were errors found by manual testing. e.g: The concurrent modification error was caught, but the page didn't reload
- 
## What I learned
 
### spring boot framework
At first it felt like complete magic. Just writing a method name in the repository and getting a working SQL query. Then I dug deeper, and learned about Hibernate, HikariCP, Spring data JPA, the spring security starter, and it all started to make sense. I still have a lot to learn but the leap in knowledge since I started working this project is huge.

### [Why entities never leave the service layer]
[What the pattern is → why it was suggested → what I understood → what would go wrong without it.]
 The pattern is the following (simplified): Vaading flow calls the service layer -> the service calls the repository -> the repository calls the postgres database.
 Simply communicating between client and database would bring severe security risks, any change is made after multiple layers of validation.
### [How Spring Security decides who can do what]
 Every AppUser has a role: USER or ADMIN. Not only can a user without the correct permission not enter a view, but it can't even call the service layer. This is done with @PreAuthorize(). 
 
### [Why database changes go through migrations]
 When changing an entity I also have to change the database, I could use Hibernates auto update, but it is unsafe and doesn't migrate data. Any changes to the database would be untracable. Flyway is a tool similar to git, but for databases. It makes it much easier to keep track of db versions.
## Stack

| Area | Technologies |
|---|---|
| Language | Java 21 |
| Framework | Spring Boot 4.1.1: Web, Data JPA, Security, Validation, Vaadin |
| Database | PostgreSQL, Flyway migrations |
| Build | Maven / Gradle |
| Testing | JUnit 5, Mockito, Testcontainers |
| Tooling | Docker Compose, GitHub Actions|
| Frontend | Vaadin components |

## Architecture
 
```mermaid
flowchart LR
    Browser[Böngésző] -->|HTTPS| Caddy[Caddy reverse proxy<br/>crm.bwsolutions.hu]
    Caddy -->|HTTP 127.0.0.1:8080| Security

    subgraph App["Spring Boot 4 + Vaadin Flow (app konténer)"]
        Security["Spring Security<br/>VaadinSecurityConfigurer + LoginView"] --> Views
        Views["Vaadin view-k (ui)<br/>Dashboard · Company · Contact · Pipeline/Deal · TierCatalog · Profile"]
        Views --> Services["Service-ek (service)<br/>Company · Contact · Deal · Tier · User · Audit<br/>@PreAuthorize"]
        Services --> Repos["Spring Data JPA repository-k (domain)<br/>Company · Contact · Deal · ServiceTier · AppUser · AuditLog"]
        Views -.->|Dashboard olvasás| Repos
        Runners["ApplicationRunner-ek<br/>AdminBootstrap · PasswordResetRunner"] --> Services
        Flyway[Flyway migrációk V1–V7] -.-> DB
    end

    Repos --> DB[("PostgreSQL 17<br/>(postgres konténer)")]
    DB -.->|napi backup.sh| OOS[(Oracle Object Storage)]
```
## Data model (részleges)
 
```mermaid
erDiagram
    APP_USER ||--o{ COMPANY : owns
    COMPANY ||--o{ CONTACT : has

    APP_USER {
        bigint id PK
        string username
        enum role
    }
    COMPANY {
        bigint id PK
        string name
        string tax_number
        enum status
        bigint owner_id FK
    }
    CONTACT {
        bigint id PK
        bigint company_id FK
        string name
        string email
        boolean is_primary
    }
```

## Screenshots
 
| Company List | Client list | Pipeline |
|---|---|---|
| ![Company List](screenshots/companies.png) | ![Client list](contacts/clients.png) | ![Pipeline](screenshots/pipeline.png) |
 
## Testing
 
I unit-test the service layer with Mockito and run integration tests against a real database with Testcontainers.
CI runs the full test suite on every push.
 
## What I'd do differently

- I'd take the leap a bit slower. At first I coded a dummy crm by hand, and underestimated how much more a production grade software actually was. It was very overwhelming at first.
- Start sooner. This journey has taught me so much and is very exciting, I only wish that I started sooner.

Majer Balázs, Computer Science BSc student at ELTE IK.
[LinkedIn](https://www.linkedin.com/in/[your-profile]) · [you@example.com](mailto:[you@example.com])
