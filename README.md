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
