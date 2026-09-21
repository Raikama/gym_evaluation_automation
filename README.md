# gym_evaluation_automation
Sistema autônomo para instrutores e academias: elimina o retrabalho manual calculando composição corporal, deltas de progresso entre avaliações e gerando relatórios em PDF em memória via n8n e Docker.

# 🏋️‍♂️ Automated Physical Assessment System (n8n + Gotenberg + Google Workspace)

Sistema autônomo de processamento antropométrico, geração *stateless* de relatórios em PDF e entrega automatizada de métricas de evolução para alunos e instrutores de academias.

---

## 📌 Visão Geral

O projeto elimina o retrabalho de cálculo manual e compilação de fichas de avaliação física. Assim que uma avaliação é registrada via Google Forms/Sheets, o fluxo em n8n:
1. Extrai as medidas antropométricas e dobras cutâneas.
2. Calcula densidade corporal, percentual de gordura (%BF), massa magra/gorda, IMC e RCQ.
3. Consulta o histórico anterior do aluno via `ID` para mensurar a evolução temporal (deltas).
4. Compila um documento visual em memória usando Chromium Headless (Gotenberg).
5. Dispara o relatório em PDF por e-mail com resumo comparativo no corpo da mensagem.

---

## 🏗️ Arquitetura da Solução

┌─────────────────────────────────────────────────────────────┐
│                 1. ENTRADA & PERSISTÊNCIA                  │
│   Google Forms (Celular/Desktop) ──► Google Sheets Base     │
└──────────────────────────────┬──────────────────────────────┘
                               │ (Row Added Trigger)
                               ▼
┌─────────────────────────────────────────────────────────────┐
│             2. ORQUESTRAÇÃO & NEGÓCIO (n8n)                 │
│                                                             │
│  [Google Sheets Trigger]                                    │
│             │                                               │
│             ▼                                               │
│  [Get Many Rows] (Busca histórico via 'ID do Aluno')        │
│             │                                               │
│             ▼                                               │
│  [Code Node (JavaScript)]:                                  │
│    • Petroski 4 Dobras (Densidade Corporal)                 │
│    • Equação de Siri (% Gordura, Massa Magra/Gorda)         │
│    • Cálculo de Deltas Temporais (Anterior vs. Atual)       │
│    • Geração de Estrutura HTML/CSS + Gráficos SVG           │
│    • Buffer Binário em Memória (prepareBinaryData)          │
└──────────────────────────────┬──────────────────────────────┘
                               │ POST multipart/form-data (index.html)
                               ▼
┌─────────────────────────────────────────────────────────────┐
│             3. MICROSSERVIÇO HEADLESS (Docker)              │
│  Gotenberg Service (Chromium Engine Stateless)              │
│  ──► Compila HTML/CSS/SVG em PDF binário direto na RAM      │
└──────────────────────────────┬──────────────────────────────┘
                               │ Output binário (data)
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                 4. DISPARO & NOTIFICAÇÃO                    │
│                                                             │
│  [Gmail Node (Send Email)]:                                 │
│    ├── Para o Aluno: Resumo da evolução no corpo do e-mail  │
│    │                 + PDF timbrado anexado                 │
│    └── Para o Instrutor: Resumo rápido das métricas         │
│                                                             │
│  [Módulo Agendado Independente (Schedule Trigger)]:         │
│    └── Disparo mensal autônomo (alerta de reavaliação 30d)  │
└─────────────────────────────────────────────────────────────┘
