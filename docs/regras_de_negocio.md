# RFP - Especificação de Negócio: Sistema AlphaBank

## 1. Visão Geral e Objetivo
O **AlphaBank** necessita de uma refatoração completa de sua base de dados relacional para:
- Suportar operações bancárias diárias com alta integridade transactional (OLTP).
- Fornecer trilha de auditoria e segurança para operações críticas.
- Servir como fonte primária de dados para a esteira analítica (Power BI / Engenharia de Dados).

---

## 2. Especificação Funcional por Domínio

### 2.1. Gestão de Clientes
- **Perfis de Público:** Atendimento focado em Pessoa Física (PF) e Pessoa Jurídica (PJ).
- **Dados Cadastrais Básicos:** E-mail, telefone, endereço completo e data de cadastro.
- **Pessoa Física (PF):** Requer CPF, Nome Completo e Data de Nascimento.
- **Pessoa Jurídica (PJ):** Requer CNPJ, Razão Social, Nome Fantasia e Data de Abertura.
- **Cardinalidade:** Um cliente pode possuir uma ou mais contas no banco.

### 2.2. Estrutura de Contas e Agências
- **Agências Bancárias:** Toda conta é alocada em uma agência específica (identificada por número, nome e endereço).
- **Tipos de Conta:** Conta Corrente, Conta Poupança e Conta Salário.
- **Contas Conjuntas (Corrente/Poupança):** Suporte a múltiplos titulares para a mesma conta (relação N:N), assim como um cliente pode ter múltiplas contas.
- **Regra Rígida de Conta Salário:** 
  - Titularidade única e restrita à Pessoa Física (PF).
  - Vinculação obrigatória a um CNPJ Empregador cadastrado na base.
  - Apenas o empregador vinculado tem permissão para realizar créditos/depósitos nessa conta.

### 2.3. Movimentações e Transações Bancárias
- **Tipos de Operação:** Registro de PIX, TED, DOC, Saques e Depósitos.
- **Atributos da Transação:** Conta de origem, conta de destino (quando aplicável), valor, timestamp exato e status (*Concluída*, *Cancelada*, *Pendente*).
- **Canais de Atendimento:** Registro do canal utilizado (*App Mobile*, *Internet Banking*, *Caixa Eletrônico*, *Agência Física*).
- **Ecossistema PIX:** Suporte a múltiplas chaves associadas à conta (CPF, CNPJ, E-mail, Telefone ou Chave Aleatória/EVP).

### 2.4. Produtos de Crédito (Cartões, Empréstimos e Boletos)
- **Cartões:**
  - Tipos: Débito, Crédito e Múltiplo.
  - Cartão de Crédito requer limite total, limite disponível, dia de vencimento e geração de faturas mensais.
  - Faturas agrupam lançamentos mensais com status (*Aberta*, *Fechada*, *Paga*).
- **Empréstimos:**
  - Atributos: Valor total, taxa de juros, quantidade de parcelas e status (*Ativo*, *Quitado*, *Inadimplente*).
  - Exige histórico detalhado de parcelas.
- **Boletos:**
  - Vinculados a uma conta recebedora.
  - Contém valor, data de emissão, data de vencimento, código de barras/linha digitável e status de pagamento.

### 2.5. Estrutura Organizacional e RH
- **Alocação:** Funcionários são vinculados a uma agência bancária.
- **Dados do Funcionário:** CPF, Nome, Cargo, Salário e Data de Admissão.
- **Hierarquia Interna:** Auto-relacionamento para representação de supervisão/gerência (`cpf_supervisor` apontando para `cpf_funcionario`).

### 2.6. Engenharia, Auditoria e Analytics
- **Fechamento de Saldo Diário:** Tabela agregada `saldo_diario` (chave primária composta `id_conta` + `dt_registro`) para otimização de consultas no Power BI sem varredura do histórico transacional.
- **Trilha de Auditoria:** Registro de alterações críticas (`log_auditoria`, `log_cliente`, `log_funcionario`) registrando autor, objeto alterado, ação, valores antigos/novos e timestamp.


