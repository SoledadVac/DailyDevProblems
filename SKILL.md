# Java Backend Developer Skill Profile

## Overview
I am a professional Java backend engineer specializing in building scalable, cloud-native applications using modern Spring ecosystems. I prioritize clean architecture, testability, and production observability. My work spans from API development to distributed system design, with recent focus on integrating AI capabilities (RAG, Agents) into enterprise backends.

## Technical Stack

### Core Technologies
- **Language**: Java 17
- **Framework**: Spring Boot 3.x, Spring Cloud Alibaba (Nacos, Sentinel, Gateway)
- **Persistence**: MyBatis-Plus
- **Database**: MySQL 8.0 (with ShardingSphere for sharding), Redis 7.x (caching), PostgreSQL 15+ (with pgvector for AI apps)
- **Message Queue**: Kafka for event streaming
- **Build Tool**: Maven (standard), Gradle (familiar)

### Cloud & DevOps
- **Containerization**: Docker, Kubernetes (deployment via Helm/ArgoCD)
- **Observability**: SkyWalking (APM), Prometheus + Grafana (metrics), ELK stack (logging)
- **CI/CD**: GitLab CI, Jenkins, Alibaba Cloud Effect
- **Cloud Platforms**: Alibaba Cloud (ACK, OSS, RocketMQ), familiar with Huawei Cloud/Tencent Cloud

### AI Integration (2025–2026 Focus)
- **LLM Framework**: Spring AI for building RAG and Agent systems
- **Vector DB**: pgvector (for <1M docs), Milvus (for large-scale)
- **Model APIs**: DashScope (Qwen-Max/Qwen-Plus), Baidu Qianfan
- **Skill Executor**: Custom Java-based skill runtime with annotation-driven tool registration

## Capabilities

### What I Can Do
✅ Design and implement RESTful/gRPC microservices  
✅ Build secure, paginated, and validated Spring Boot APIs  
✅ Integrate Redis for caching, distributed locks, and rate limiting  
✅ Implement distributed transactions using Seata or Saga pattern  
✅ Write unit/integration tests with JUnit 5, Mockito, Testcontainers  
✅ Containerize apps with multi-stage Docker builds  
✅ Add observability via Micrometer + SkyWalking  
✅ Develop RAG applications using LangChain4j + pgvector  
✅ Create extensible Agent Skill executors in Java  

### What I Avoid / Limitations
❌ Frontend development (Vue/React) — I delegate to frontend engineers  
❌ Writing raw SQL for complex analytics — prefer MyBatis-Plus or JOOQ  
❌ Direct shell/script execution — all external actions must go through safe HTTP/tool interfaces  
❌ Using deprecated tech (e.g., Spring Boot 2.x, Hystrix, Eureka without migration plan)

## Preferred Patterns
- **Layered Architecture**: Controller → Service → Repository
- **DTOs** for API input/output (never expose entities directly)
- **Validation**: Bean Validation (`@Valid`) + custom validators
- **Error Handling**: `@ControllerAdvice` with standardized error codes
- **Configuration**: Externalized via Nacos or `application.yml`
- **Security**: Spring Security + JWT/OAuth2 (not basic auth)

## Example Tools I Use
- **Local Dev**: IntelliJ IDEA + Tongyi Lingma plugin
- **API Testing**: Apifox / Postman
- **Database**: DataGrip / DBeaver
- **AI Coding**: Trae, GitHub Copilot, Tongyi Lingma

## Invocation Guidance for AI Agent
When generating code or advice for me:
- Assume **Spring Boot 3.2+** and **JDK 17+**
- Use **MyBatis-Plus** for data access unless specified otherwise
- Prefer **pgvector over Milvus** for small-to-medium RAG projects
- Always include **input validation**, **error handling**, and **logging**
- For AI features, use **LangChain4j** with **DashScope Qwen models**
- Never suggest Python scripts — I work in pure Java ecosystem
