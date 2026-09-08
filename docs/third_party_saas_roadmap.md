# Segundo projeto — SaaS de Gestão de Terceiros e Aprovação B2B

## 1. Visão geral

Este projeto será uma **plataforma SaaS B2B para gestão do ciclo de vida de terceiros**, com foco inicial em:

- cadastro de terceiros;
- solicitação e coleta de documentos;
- cobrança automática de pendências;
- análise e aprovação documental;
- workflows de aprovação;
- controle de validade;
- renovação de documentos;
- trilha de auditoria;
- homologação de fornecedores e prestadores.

O objetivo é construir primeiro um **backend robusto e production-oriented**, sem depender de IA como peça central.

A IA pode entrar depois como feature de apoio, por exemplo em classificação documental, extração de dados e análise assistida.

---

## 2. Problema

Empresas que contratam fornecedores, prestadores e parceiros frequentemente controlam documentação e homologação por uma combinação de:

- planilhas;
- e-mails;
- WhatsApp;
- pastas compartilhadas;
- sistemas internos desconectados.

Isso cria problemas como:

- documentos faltantes;
- documentos vencidos;
- dificuldade em saber o status de cada terceiro;
- cobrança manual;
- pouca rastreabilidade;
- aprovações informais;
- perda de histórico;
- dificuldade para provar quem aprovou o quê;
- falta de padronização;
- gargalos no onboarding.

O sistema deve centralizar esse processo.

---

## 3. Proposta de valor

A plataforma permite que uma empresa transforme isto:

```text
Planilha
   +
Email
   +
WhatsApp
   +
Drive
   +
Aprovações informais
```

em:

```text
Third Party
     ↓
Document Requirements
     ↓
Collection
     ↓
Validation
     ↓
Approval Workflow
     ↓
Homologation
     ↓
Monitoring
     ↓
Renewal
```

---

## 4. Escopo inicial do produto

O produto completo pode se tornar grande.

Por isso, a primeira versão deve ter um recorte específico:

> **Coleta, cobrança, análise e aprovação de documentos de terceiros.**

Não começar construindo um Vendor Management System completo.

A expansão deve ser incremental.

---

## 5. Ciclo de vida do terceiro

```text
Created
   ↓
Invited
   ↓
Documentation Pending
   ↓
Documentation Submitted
   ↓
Under Review
   ↓
Approval Process
   ↓
Approved / Rejected
   ↓
Active
   ↓
Monitoring
   ↓
Renewal Required
   ↓
Revalidation
```

Posteriormente:

```text
Active
   ↓
Suspended
   ↓
Offboarding
   ↓
Inactive
```

---

## 6. Principais usuários

### 6.1 Empresa contratante

Usuários internos da organização.

Exemplos:

- comprador;
- analista;
- compliance;
- jurídico;
- financeiro;
- gestor;
- administrador.

### 6.2 Terceiro

Usuário externo.

Pode ser:

- fornecedor;
- prestador de serviço;
- parceiro;
- clínica;
- consultoria;
- empresa terceirizada.

O terceiro deve acessar apenas os próprios dados e solicitações.

---

## 7. Estrutura multi-tenant

Desde cedo, o domínio deve considerar SaaS multi-tenant.

```text
Platform
│
├── Organization A
│   ├── Users
│   ├── Third Parties
│   ├── Workflows
│   └── Documents
│
├── Organization B
│   ├── Users
│   ├── Third Parties
│   ├── Workflows
│   └── Documents
│
└── Organization C
```

Nenhuma organização pode acessar dados de outra.

Esse isolamento deve ser tratado como requisito arquitetural, não apenas como filtro de frontend.

---

## 8. Domínio principal

Entidades iniciais:

```text
Organization
User
Membership
Role
Permission

ThirdParty
ThirdPartyContact

DocumentType
DocumentRequirement
DocumentRequest
Document
DocumentVersion

Review
ReviewDecision

Workflow
WorkflowStep
WorkflowExecution
Approval

Notification

AuditLog
```

Posteriormente:

```text
Contract
RiskAssessment
Integration
Webhook
ApiKey
Subscription
Usage
```

---

## 9. Exemplo de estrutura

```text
Organization
│
├── Users
│
├── Third Parties
│   │
│   ├── Company A
│   │   ├── Contacts
│   │   ├── Requirements
│   │   ├── Documents
│   │   └── Approvals
│   │
│   └── Company B
│
├── Document Templates
│
├── Approval Workflows
│
├── Notifications
│
└── Audit Logs
```

---

## 10. Status documentais

Um documento pode passar por:

```text
REQUESTED
    ↓
PENDING
    ↓
UPLOADED
    ↓
UNDER_REVIEW
    ↓
┌─────────────┐
│             │
APPROVED    REJECTED
                ↓
            RESUBMITTED
                ↓
          UNDER_REVIEW
```

Também:

```text
APPROVED
    ↓
EXPIRING
    ↓
EXPIRED
    ↓
RENEWAL_REQUIRED
```

Evitar representar tudo com um único boolean como:

```python
approved = True
```

O domínio exige estados explícitos.

---

## 11. MVP — versão 0.1

O primeiro MVP deve provar o fluxo principal.

### Funcionalidades

- criação de organização;
- criação de usuários;
- autenticação;
- cadastro de terceiros;
- criação de tipos de documentos;
- atribuição de documentos obrigatórios;
- upload de documentos;
- download;
- status;
- aprovação;
- rejeição;
- comentários;
- histórico;
- datas de validade.

Fluxo:

```text
Admin
  ↓
creates Third Party
  ↓
assigns document requirements
  ↓
Third Party uploads documents
  ↓
Analyst reviews
  ↓
Approve / Reject
```

Esse é o primeiro vertical slice.

---

## 12. V0.2 — notificações e cobrança

Adicionar:

- notificações;
- lembretes;
- documentos pendentes;
- documentos próximos do vencimento;
- documentos vencidos;
- reenvio solicitado.

Exemplo:

```text
Document expires in 30 days
        ↓
Event generated
        ↓
Background job
        ↓
Notification
        ↓
Third party receives reminder
```

---

## 13. V0.3 — templates

Empresas não devem cadastrar exigências individualmente para cada fornecedor.

Criar templates:

```text
Template: Software Supplier

Required documents
├── CNPJ
├── Contract
├── Tax Certificate
├── Information Security Questionnaire
└── Privacy Agreement
```

Outro:

```text
Template: Service Provider

├── Company Registration
├── Insurance
├── Certificate A
└── Certificate B
```

Ao cadastrar um terceiro:

```text
Third Party
     +
Template
     ↓
Requirements generated
```

---

## 14. V0.4 — workflows de aprovação

Depois adicionar aprovação em múltiplas etapas.

Exemplo:

```text
Document
   ↓
Analyst
   ↓
Compliance
   ↓
Manager
   ↓
Approved
```

Outro:

```text
Supplier
   ↓
Procurement
   ↓
Compliance
   ↓
Legal
   ↓
Finance
   ↓
Homologated
```

---

## 15. Workflow engine

Uma das partes mais importantes tecnicamente.

Modelo inicial:

```text
Workflow
├── Step 1
├── Step 2
├── Step 3
└── Step 4
```

Execução:

```text
WorkflowExecution
│
├── current_step
├── status
├── started_at
└── completed_at
```

Posteriormente podem existir:

- branches;
- condições;
- parallel approvals;
- escalation;
- timeout;
- delegation.

Exemplo:

```text
Contract value < 50k
        ↓
Compliance
        ↓
Approved
```

versus:

```text
Contract value >= 50k
        ↓
Compliance
        ↓
Legal
        ↓
Director
```

---

## 16. V0.5 — regras condicionais

Exemplo:

```text
IF
third_party.category == "critical"
THEN
require:
    - information_security_review
    - legal_review
```

Outro:

```text
IF
contract_value > 100000
THEN
approval:
    director
```

Isso pode evoluir posteriormente para um pequeno rules engine.

---

## 17. V0.6 — audit trail

Toda ação relevante deve gerar evento.

Exemplo:

```text
09:31
Victor uploaded document

09:44
Maria started review

10:02
Maria rejected document

Reason:
Invalid expiration date

10:23
Victor uploaded new version

11:12
Maria approved document
```

Não apenas:

```text
document.updated_at
```

Precisamos de histórico explícito.

---

## 18. V0.7 — versionamento documental

Nunca substituir silenciosamente documentos.

```text
Document
│
├── Version 1
├── Version 2
└── Version 3
```

Exemplo:

```text
certificate.pdf

v1
Rejected

v2
Approved

v3
Renewal
```

---

## 19. V0.8 — webhooks

Permitir integração com sistemas externos.

Eventos:

```text
third_party.created

document.uploaded

document.approved

document.rejected

document.expiring

third_party.approved

third_party.suspended
```

Cliente cadastra:

```text
POST https://customer.example/webhooks
```

A plataforma envia eventos.

Aqui entram:

- assinatura;
- retries;
- idempotência;
- delivery logs;
- dead-letter handling.

---

## 20. V0.9 — API pública

Criar autenticação por API keys.

Exemplo:

```http
POST /api/v1/third-parties
```

```http
GET /api/v1/third-parties/{id}
```

```http
POST /api/v1/documents
```

Isso torna o produto integrável com:

- ERP;
- procurement;
- RH;
- CRM;
- sistemas internos.

---

## 21. V1.0 — SaaS de verdade

Nesse estágio entram:

- onboarding;
- planos;
- quotas;
- billing;
- usage;
- administração;
- observabilidade;
- documentação da API;
- ambiente produtivo.

---

## 22. Arquitetura inicial

Eu começaria como **modular monolith**.

Não microservices.

```text
Client
  ↓
API
  ↓
┌─────────────────────────────────────┐
│              Application            │
│                                     │
│  Identity                           │
│  Organizations                      │
│  Third Parties                      │
│  Documents                          │
│  Workflows                          │
│  Notifications                      │
│  Audit                              │
└─────────────────────────────────────┘
       │          │           │
       ↓          ↓           ↓
 PostgreSQL     Redis     Object Storage
```

---

## 23. Por que modular monolith

O projeto precisa ensinar:

- domínio;
- boundaries;
- dependency management;
- transações;
- modularidade;
- testes;
- arquitetura.

Microservices cedo adicionariam complexidade operacional sem benefício real.

O objetivo é permitir que módulos possam eventualmente ser separados, sem começar distribuído.

---

## 24. Estrutura possível do backend

```text
src/
├── identity/
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── api/
│
├── organizations/
│
├── third_parties/
│
├── documents/
│
├── workflows/
│
├── notifications/
│
├── audit/
│
├── shared/
│
└── main.py
```

Não precisa nascer exatamente assim.

A estrutura deve evoluir conforme entendermos o domínio.

---

## 25. Stack inicial

### Backend

```text
Python
FastAPI
```

### Database

```text
PostgreSQL
```

### Cache / jobs

```text
Redis
```

### Object storage

Local:

```text
MinIO
```

Produção:

```text
S3-compatible storage
```

### Containers

```text
Docker
Docker Compose
```

---

## 26. ORM

Uma opção natural:

```text
SQLAlchemy
```

Mais:

```text
Alembic
```

para migrations.

---

## 27. Jobs assíncronos

Casos:

```text
send email
expire document
generate reminder
deliver webhook
process file
```

Não colocar essas operações dentro da request HTTP.

Arquitetura:

```text
API
 ↓
Database
 ↓
Queue
 ↓
Worker
```

---

## 28. Banco de dados

Exemplo inicial:

```text
organizations

users
memberships

third_parties
third_party_contacts

document_types
document_requirements
documents
document_versions

reviews

workflows
workflow_steps
workflow_executions
approvals

notifications

audit_logs
```

---

## 29. Multi-tenancy

Começar provavelmente com:

```text
organization_id
```

nas entidades pertencentes a tenants.

Exemplo:

```text
documents
├── id
├── organization_id
├── third_party_id
└── ...
```

Toda query tenant-aware deve respeitar:

```text
current_organization
```

Isso deve ser centralizado para reduzir risco de vazamento entre tenants.

---

## 30. RBAC

Exemplo inicial:

```text
OWNER
ADMIN
ANALYST
REVIEWER
MEMBER
```

Usuário externo:

```text
THIRD_PARTY
```

Posteriormente migrar para permissões mais granulares:

```text
third_party:create
third_party:view

document:view
document:review

workflow:create
workflow:approve
```

---

## 31. Autenticação

Fases iniciais:

```text
email
password
access token
refresh token
```

Posteriormente:

```text
SSO
OAuth
SAML
MFA
```

Não começar implementando identidade enterprise completa.

---

## 32. Segurança

Requisitos importantes:

- password hashing;
- token expiration;
- authorization server-side;
- tenant isolation;
- secure file access;
- signed URLs;
- rate limiting;
- input validation;
- secret management;
- audit trail;
- malware/file validation posteriormente.

---

## 33. Upload de arquivos

Nunca salvar arquivo grande diretamente no PostgreSQL.

```text
Client
   ↓
Object Storage
   ↓
metadata
   ↓
PostgreSQL
```

Banco guarda:

```text
storage_key
filename
mime_type
size
checksum
created_at
```

---

## 34. Integridade de arquivos

Gerar hash:

```text
SHA-256
```

Isso permite:

- detectar alteração;
- identificar duplicação;
- garantir integridade;
- melhorar auditoria.

---

## 35. Idempotência

Importante em endpoints como:

```text
POST /third-parties
POST /documents
POST /payments
POST /workflow-executions
```

Cliente pode reenviar uma request após timeout sem criar duplicação.

---

## 36. Transactions

Exemplo:

```text
approve document
     ↓
update status
     +
create approval
     +
create audit event
     +
generate domain event
```

Isso deve preservar consistência.

---

## 37. Domain events

Exemplo:

```python
DocumentApproved
DocumentRejected
DocumentExpired
ThirdPartyApproved
```

Podem causar:

```text
DocumentApproved
       ↓
Audit Log
       ↓
Notification
       ↓
Workflow advancement
       ↓
Webhook
```

---

## 38. Eventual consistency

Nem tudo precisa acontecer dentro da mesma transação.

Exemplo:

```text
Document Approved
       ↓
COMMIT
       ↓
event
       ↓
email
       ↓
webhook
```

Falha no email não deve desfazer aprovação documental.

---

## 39. Outbox Pattern

Posteriormente, podemos implementar:

```text
Database Transaction
       ↓
business change
       +
outbox event
       ↓
COMMIT
       ↓
worker
       ↓
external systems
```

Isso é um ótimo tópico de arquitetura para o projeto.

---

## 40. Notificações

Canais possíveis:

```text
In-app
Email
Webhook
```

Posteriormente:

```text
WhatsApp
Slack
Teams
```

---

## 41. Scheduler

Necessário para:

```text
documents expiring in 30 days
documents expiring in 7 days
expired documents
overdue requests
workflow SLA exceeded
```

---

## 42. Audit log

Modelo:

```text
actor
action
resource_type
resource_id
timestamp
metadata
```

Exemplo:

```json
{
  "action": "document.rejected",
  "resource_type": "document",
  "actor": "user_123",
  "reason": "expired_document"
}
```

---

## 43. Observabilidade

Desde relativamente cedo:

```text
Logs
Metrics
Tracing
```

Logs estruturados.

Exemplo:

```text
request_id
organization_id
user_id
operation
duration
status
```

---

## 44. Error handling

Evitar retornar erros inconsistentes.

Exemplo:

```json
{
  "code": "DOCUMENT_ALREADY_APPROVED",
  "message": "Document cannot be modified after approval."
}
```

---

## 45. Testes

### Unit

Regras de domínio.

```text
Document cannot be approved twice.
```

```text
Expired document cannot keep ACTIVE status.
```

### Integration

```text
API
 ↓
PostgreSQL
```

### Authorization

Essenciais.

```text
Organization A
cannot access
Organization B
```

### Workflow

```text
step 1 approved
→ step 2 becomes active
```

### End-to-end

Posteriormente:

```text
create third party
→ request docs
→ upload
→ review
→ approve
→ homologate
```

---

## 46. CI/CD

Pipeline inicial:

```text
Push
 ↓
Lint
 ↓
Tests
 ↓
Build
 ↓
Docker Image
```

Posteriormente:

```text
Deploy staging
 ↓
Integration tests
 ↓
Production
```

---

## 47. Ambientes

```text
local
testing
staging
production
```

Configuração via environment variables.

---

## 48. API versioning

Desde cedo:

```text
/api/v1/
```

Não porque teremos v2 imediatamente, mas para evitar acoplamento futuro desnecessário.

---

## 49. Paginação

Evitar:

```http
GET /third-parties
```

retornando tudo.

Usar paginação.

Eventualmente cursor-based para grandes datasets.

---

## 50. Search

Inicial:

```text
PostgreSQL
```

Posteriormente:

```text
full-text search
```

Só adicionar Elasticsearch/OpenSearch se existir necessidade real.

---

## 51. Frontend

O backend é o foco inicial.

Quando entrar frontend:

```text
Next.js
React
TypeScript
```

Portais diferentes podem existir:

```text
Internal Portal

Third-Party Portal
```

Mas podem compartilhar o mesmo app inicialmente.

---

## 52. Portal interno

Dashboard:

```text
Third Parties

Total       342
Approved    250
Pending      53
Blocked      12
Expiring     27
```

---

## 53. Portal do terceiro

Interface simplificada:

```text
Required Documents

✓ Company Registration

✓ Certificate A

⚠ Certificate B
  expires in 14 days

✕ Certificate C
  rejected

○ Document D
  pending upload
```

---

## 54. Billing

Só depois que o produto estiver funcional.

Possíveis métricas:

```text
number of third parties
number of active users
documents processed
```

Exemplo:

```text
Starter
100 third parties

Professional
1,000 third parties

Enterprise
Custom
```

---

## 55. Integrações

Posteriormente:

```text
ERP
HR systems
Procurement
Email
Cloud Storage
CRM
Identity Providers
```

---

## 56. IA — somente posteriormente

A IA deve resolver problemas específicos.

Não adicionar LLM apenas para dizer que o produto tem IA.

---

## 57. Classificação documental

Entrada:

```text
PDF
```

Saída:

```json
{
  "document_type": "tax_certificate",
  "confidence": 0.97
}
```

---

## 58. Extração

Exemplo:

```text
Certificate
    ↓
CNPJ
Company Name
Issue Date
Expiration Date
```

Depois um humano pode confirmar.

---

## 59. Validação

Combinar IA com regras determinísticas:

```text
AI extraction
     ↓
structured fields
     ↓
rules
```

Exemplo:

```text
document.cnpj == third_party.cnpj
```

---

## 60. Human-in-the-loop

Nunca fazer:

```text
AI
 ↓
automatic irreversible approval
```

Preferir:

```text
AI
 ↓
suggestion
 ↓
human review
 ↓
decision
```

principalmente para decisões relevantes de compliance.

---

## 61. Contratos

Expansão posterior:

```text
Contract
├── parties
├── value
├── start date
├── end date
├── renewal
├── documents
└── obligations
```

---

## 62. Risk management

Posteriormente:

```text
Third Party
    ↓
Risk Assessment
    ↓
LOW
MEDIUM
HIGH
CRITICAL
```

Isso pode influenciar workflows.

---

## 63. Questionários

Exemplo:

```text
Information Security Assessment

Privacy Assessment

Compliance Assessment

Financial Assessment
```

---

## 64. Reavaliação periódica

```text
Third Party approved
       ↓
12 months
       ↓
Reassessment
       ↓
new documents
       ↓
new approval
```

---

## 65. Offboarding

Fim do relacionamento:

```text
Contract terminated
       ↓
Offboarding
       ↓
revoke access
       ↓
close requirements
       ↓
archive relationship
```

---

## 66. Produto completo no futuro

Visão de longo prazo:

```text
Third-Party Lifecycle Management

├── Intake
├── Onboarding
├── Documentation
├── Due Diligence
├── Approval
├── Contract
├── Compliance
├── Risk
├── Monitoring
├── Renewal
├── Reassessment
└── Offboarding
```

---

## 67. O que NÃO construir no início

Evitar:

```text
microservices
Kafka
Kubernetes
GraphQL
CQRS completo
event sourcing
complex rules engine
custom identity provider
AI agents
blockchain
```

A não ser que uma necessidade real apareça.

---

## 68. Filosofia arquitetural

O projeto deve seguir:

> **Complexidade deve ser conquistada por necessidade, não adicionada antecipadamente.**

Começar com:

```text
FastAPI
PostgreSQL
Redis
Object Storage
Worker
```

e evoluir.

---

## 69. Objetivos de aprendizado

Este projeto deve aprofundar:

### Backend Engineering

- HTTP;
- REST;
- auth;
- databases;
- transactions;
- caching;
- queues;
- background jobs;
- file storage;
- concurrency;
- APIs.

### Software Architecture

- modular monolith;
- boundaries;
- domain modeling;
- dependency inversion;
- domain events;
- transactional boundaries;
- eventual consistency;
- outbox;
- idempotency.

### SaaS

- multi-tenancy;
- RBAC;
- subscriptions;
- usage;
- organizations;
- external users;
- API keys;
- webhooks.

### Production Engineering

- Docker;
- CI/CD;
- testing;
- logging;
- tracing;
- metrics;
- deployment;
- migrations.

---

## 70. Ordem de implementação sugerida

```text
Phase 0
Domain exploration
        ↓
Phase 1
Project foundation
        ↓
Phase 2
Authentication + Organizations
        ↓
Phase 3
Third Parties
        ↓
Phase 4
Document Requirements
        ↓
Phase 5
Document Upload
        ↓
Phase 6
Review / Approval
        ↓
Phase 7
Audit Log
        ↓
Phase 8
Notifications
        ↓
Phase 9
Expiration / Renewal
        ↓
Phase 10
Templates
        ↓
Phase 11
Workflow Engine
        ↓
Phase 12
Webhooks
        ↓
Phase 13
Public API
        ↓
Phase 14
Frontend
        ↓
Phase 15
Billing
        ↓
Phase 16
Production deployment
        ↓
Phase 17
AI features
```

---

## 71. Primeira milestone publicável

```text
Organization
      ↓
Third Party
      ↓
Document Requirement
      ↓
Upload
      ↓
Review
      ↓
Approve / Reject
      ↓
Audit Trail
```

Se isso estiver sólido, testado e documentado, já existe um projeto coerente de portfólio.

---

## 72. Roadmap resumido

```text
V0.1
Document lifecycle

V0.2
Notifications

V0.3
Templates

V0.4
Approval workflows

V0.5
Conditional rules

V0.6
Audit + document versioning

V0.7
Expiration + renewals

V0.8
Webhooks

V0.9
Public API

V1.0
Deployable SaaS

V1.x
Frontend + billing + integrations

V2
Risk / contracts / assessments

V3
AI-assisted document intelligence
```

---

## 73. Possíveis nomes

Não decidir ainda.

Algumas direções de naming:

```text
vendor lifecycle
third-party management
vendor compliance
document compliance
partner onboarding
supplier governance
```

O nome deve ser escolhido depois de uma análise mínima de mercado e domínio disponível.

---

## 74. Estrutura futura do repositório

```text
project/
├── AGENTS.md
├── README.md
├── pyproject.toml
├── docker-compose.yml
├── .env.example
│
├── src/
│
├── tests/
│
├── migrations/
│
├── docs/
│   ├── architecture/
│   ├── domain/
│   ├── decisions/
│   ├── study/
│   └── experiments/
│
└── scripts/
```

Assim como no `medaudit`, `docs/` pode funcionar como seu vault do Obsidian.

---

## 75. ADRs

Decisões arquiteturais importantes devem ser documentadas.

Exemplo:

```text
docs/decisions/

001-modular-monolith.md
002-postgresql.md
003-multi-tenancy-strategy.md
004-object-storage.md
005-background-job-strategy.md
006-domain-events.md
```

Cada ADR deve registrar:

```text
Context
Decision
Alternatives
Consequences
```

---

## 76. README público

Quando o projeto estiver apresentável, o README deve explicar:

```text
What problem does this solve?

Architecture

Core domain

Technology stack

How to run

Testing

API

Design decisions

Roadmap
```

Não transformar README em badge wall.

---

## 77. Métricas técnicas

Posteriormente acompanhar:

```text
request latency
error rate
queue latency
job failures
webhook delivery success
database connection usage
document processing time
```

---

## 78. Métricas de produto

Se virar SaaS:

```text
third parties onboarded
average onboarding time
document rejection rate
pending documents
expired documents
approval time
workflow completion time
```

---

## 79. Possível diferencial futuro

Um possível posicionamento:

> **Third-party onboarding and compliance workflows without spreadsheets and endless email follow-ups.**

A proposta não precisa ser:

> "IA para gestão de fornecedores."

IA pode ser tecnologia interna.

A dor continua sendo:

> reduzir trabalho operacional e aumentar rastreabilidade.

---

## 80. Critério de sucesso do projeto

Mesmo que nunca vire uma empresa, o projeto terá cumprido seu papel se demonstrar claramente que você consegue construir:

```text
a multi-tenant B2B system
        +
complex domain rules
        +
reliable backend
        +
asynchronous processing
        +
security / permissions
        +
production architecture
```

Se depois houver sinais de mercado, ele pode continuar evoluindo para um SaaS real.

---

## 81. Relação com os outros projetos

```text
medaudit
    ↓
AI Engineering / RAG

Third-Party SaaS
    ↓
Backend / Architecture / SaaS

Data Platform
    ↓
Data Engineering

coping_struggles_prediction v2
    ↓
ML Engineering / MLOps
```

O SaaS pode posteriormente absorver o projeto full-stack:

```text
Backend
   ↓
Frontend
   ↓
Billing
   ↓
Integrations
   ↓
Production
   ↓
AI features
```

Assim não precisamos criar um quinto projeto artificial apenas para provar full-stack.

---

## 82. Regra principal

Este projeto não deve ser desenvolvido como checklist de tecnologias.

Não fazer:

```text
"preciso colocar Redis"
"preciso colocar Kafka"
"preciso usar microservices"
```

Fazer:

```text
problema
   ↓
requisito
   ↓
trade-offs
   ↓
solução
   ↓
measurement
```

A tecnologia entra porque resolve um problema concreto.

---

## 83. Próximo passo quando o `medaudit` estiver em V1

Antes de criar o repositório:

```text
1. Pesquisa de mercado
2. Definir ICP
3. Definir problema inicial
4. Mapear concorrentes
5. Escolher wedge
6. Modelar domínio
7. Escrever requirements
8. Desenhar arquitetura inicial
9. Criar repository
10. Implementar primeiro vertical slice
```

A primeira linha de código só vem depois de sabermos **para quem estamos construindo e qual problema específico estamos resolvendo**.

---

## 84. Wedge inicial recomendado

Hoje, o melhor candidato é:

> **Coleta, cobrança e aprovação documental de terceiros para empresas B2B.**

Não:

> plataforma completa de gestão de terceiros.

A ideia é entrar pelo problema pequeno e expandir:

```text
Documents
    ↓
Approvals
    ↓
Renewals
    ↓
Workflows
    ↓
Compliance
    ↓
Risk
    ↓
Contracts
    ↓
Third-Party Lifecycle Platform
```

Esse será o princípio norteador do segundo projeto.
