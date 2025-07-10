# RELATÓRIO DE IMPLEMENTAÇÃO DE SERVIÇOS AWS

---

**Data:** 10/07/2025  
**Empresa:** Abstergo Industries  
**Responsável:** Filipi Firmino

---

## 📑 Introdução

Este relatório apresenta o processo de implementação de ferramentas na empresa **Abstergo Industries**, realizado por Filipi Firmino. O objetivo do projeto foi **eleger 3 serviços AWS** para promover a diminuição de custos imediatos.

---

## 🛠️ Descrição do Projeto

O projeto foi dividido em **3 etapas**, cada uma com objetivos específicos. A seguir, as etapas detalhadas:

---

### Etapa 1: Amazon S3 + S3 Glacier

- **Armazenamento escalável e de baixo custo**
- Utilização para processar pedidos, comunicação com sistemas parceiros, validação de XMLs e integração com APIs externas de fornecedores.
- Eliminação de servidores ligados 24/7: Lambda executa sob demanda, sem custos de ociosidade.

**Passo a passo da implementação:**

1. Criar buckets no Amazon S3 separados por tipo de dado:
   - `notas-fiscais/`, `relatorios/`, `logs-api/`, etc.
2. Definir **políticas de ciclo de vida** para mover arquivos pouco acessados para o **S3 Glacier** após 30 dias.
3. Habilitar versionamento e compressão automática para reduzir duplicidade.
4. Utilizar **S3 Storage Lens** para analisar padrões de acesso e automatizar otimizações.

**Por que reduz custos?**
- Não há custo fixo de instância 24h.
- Pagamento por segundo de uso real (ótimo para workloads intermitentes).
- Redução do consumo excessivo de conexões com RDS Proxy.

---

### Etapa 2: AWS Lambda

- **Execução de aplicações e integrações sob demanda (serverless)**
- Processamento de pedidos, comunicação com parceiros, validação de XMLs e integração com APIs externas.
- Lambda executa apenas quando necessário, eliminando custos de ociosidade.

**Passo a passo da implementação:**

1. Criar **funções Lambda** para:
   - Processar pedidos enviados via API.
   - Validar e salvar arquivos XML no S3.
   - Consumir APIs de parceiros comerciais.
2. Integrar Lambda com API Gateway para receber requisições HTTP.
3. Disparar funções com **eventos automáticos do S3** ou **mensagens do SQS/SNS** (ex: após upload de nota fiscal).
4. Monitorar execuções com **CloudWatch Logs** e **X-Ray** para análise de desempenho e erros.

**Por que reduz custos?**
- Cobrança apenas pelo tempo de execução (em milissegundos).
- Eliminação da necessidade de EC2 ligadas o tempo todo.
- Escalabilidade automática conforme demanda, sem superdimensionamento.

---

### Etapa 3: Amazon Aurora Serverless v2 (com RDS Proxy)

- **Banco de dados relacional escalável e sob demanda**
- Armazenamento de cadastros, estoque, pedidos e autenticação.
- Aurora Serverless v2 escala automaticamente e cobra apenas pelos recursos consumidos.
- Integração com **RDS Proxy** para melhor reutilização de conexões e redução de picos de consumo.

**Passo a passo da implementação:**

1. Criar instância **Aurora Serverless v2** (MySQL ou PostgreSQL).
2. Habilitar **auto-scaling** com capacidade mínima e máxima conforme a carga.
3. Configurar **RDS Proxy** para compartilhamento de conexões entre funções Lambda.
4. Usar **CloudWatch** para monitoramento e alertas de pico.
5. Automatizar snapshots e backups com política de retenção baseada em compliance.

**Por que reduz custos?**
- Não há custo fixo de instância 24h.
- Pagamento por segundo de uso real.
- Redução do consumo excessivo de conexões com RDS Proxy.

---

## ✅ Conclusão

A implementação das ferramentas na *Abstergo Industries* visa:
- Cobrança apenas por tempo de execução ou uso.
- Escalabilidade automática conforme demanda.
- Redução de custos com armazenamento e conexões.
- Aumento da eficiência e produtividade.

**Recomendação:**
- Continuar utilizando as ferramentas implementadas.
- Buscar novas tecnologias para aprimorar ainda mais os processos da empresa.

---

## 📎 Anexos

### Diagrama da Arquitetura

```
                     +-----------------------+
                     |   Usuários/Parceiros  |
                     +----------+------------+
                                |
                                v
                      +------------------+
                      |  API Gateway     |
                      +------------------+
                                |
                                v
                     +-----------------------+
                     |  AWS Lambda Functions |
                     | - Processa pedidos    |
                     | - Valida XMLs         |
                     | - Integra APIs        |
                     +----------+------------+
                                |
        +-----------------------+--------------------------+
        |                      |                          |
        v                      v                          v
 +-------------+      +----------------+        +----------------------+
 | Amazon S3   |<---->| Amazon S3      |        | Amazon Aurora        |
 | (upload XML)|      | Lifecycle Rules|        | Serverless v2 + RDS  |
 +-------------+      | -> Glacier     |        | Proxy                |
                      +----------------+        +----------------------+
                                                       |
                                                       v
                                               +---------------+
                                               | CloudWatch/X-Ray|
                                               +----------------+
```

**Legenda:**
- 🔷 **Amazon S3:** Armazena arquivos como notas fiscais, XMLs, backups.
- 🔷 **Lambda:** Processa eventos sob demanda.
- 🔷 **Aurora Serverless v2:** Armazena dados operacionais e comerciais.
- 🔷 **S3 Glacier:** Arquivos antigos movidos automaticamente.

---

### 🛡️ Plano de Contingência e Escalabilidade

**Objetivo:** Garantir alta disponibilidade e recuperação rápida com o menor custo possível.

#### Backup & Recuperação

- **Amazon S3:** Versionamento ativado para evitar perda de dados em sobrescritas.
- **S3 Glacier:** Armazena dados antigos por compliance/regulamentação.
- **Aurora:**
  - Snapshots automáticos diários.
  - Retenção de backups por 15 a 30 dias.
  - Failover automático em caso de falha do nó principal.

#### Alta Disponibilidade

- **Aurora Serverless v2:** Multi-AZ com failover embutido.
- **Lambda + API Gateway:** Operam em múltiplas zonas de disponibilidade automaticamente.
- **S3:** Redundante em 3 zonas de disponibilidade por padrão.

#### Escalabilidade Automática

- **Lambda:** Escala automaticamente por evento.
- **Aurora Serverless v2:** Escala de forma granular conforme demanda (vCPU e memória).
- **API Gateway:** Escala automaticamente para milhões de requisições.

#### Testes de Recuperação

- Testes mensais de restauração de backup do Aurora em ambiente separado.
- Simulação de falha de função Lambda com monitoramento via CloudWatch e alarmes.
- Uso do AWS Well-Architected Tool para avaliação periódica de resiliência.

---

## 📝 Script Terraform (versão simplificada)

Abaixo, uma **estrutura básica** com foco nos principais recursos:

```hcl
provider "aws" {
  region = "us-east-1"
}

# Bucket S3 com lifecycle
resource "aws_s3_bucket" "med_docs" {
  bucket = "farmaceutica-documentos"

  lifecycle_rule {
    enabled = true

    transition {
      days          = 30
      storage_class = "GLACIER"
    }
  }

  versioning {
    enabled = true
  }
}

# Função Lambda básica
resource "aws_lambda_function" "processa_pedido" {
  filename         = "lambda.zip"
  function_name    = "processaPedido"
  role             = aws_iam_role.lambda_exec.arn
  handler          = "index.handler"
  runtime          = "nodejs18.x"
  source_code_hash = filebase64sha256("lambda.zip")
}

resource "aws_iam_role" "lambda_exec" {
  name = "lambda_exec_role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [{
      Action = "sts:AssumeRole",
      Effect = "Allow",
      Principal = {
        Service = "lambda.amazonaws.com"
      }
    }]
  })
}

# Aurora Serverless Cluster
resource "aws_rds_cluster" "aurora_farmaceutica" {
  engine         = "aurora-mysql"
  engine_mode    = "provisioned" # substitua por 'serverless' se for v1
  cluster_identifier = "aurora-farmaceutica"
  master_username    = "admin"
  master_password    = "SenhaForte123"
  skip_final_snapshot = true
}
```

---

**Assinatura do responsável pelo projeto:**  
Filipi Firmino
