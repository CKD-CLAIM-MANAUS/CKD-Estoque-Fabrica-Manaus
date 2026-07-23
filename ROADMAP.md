# SRMV — Sistema de Rastreabilidade e Movimentação de Veículos

> Evolução do app atual **Estoque Fábrica Manaus** (PWA + Firebase) para um WMS
> completo de fábrica de motocicletas e ATVs. Documento vivo — nosso mapa de trabalho.

## Visão (extraída da conversa de referência)

- **Nome:** SRMV — Sistema de Rastreabilidade e Movimentação de Veículos.
- **Objetivo:** rastreabilidade completa (VIN/chassi), movimentação logística e expedição.
- **Estrutura física:** Galpão → Rua → Baia → Posição.
- **Módulos previstos:** Dashboard, Mapa do Estoque, Veículos, Movimentações,
  Pedidos, Concessionárias, Relatórios, Auditoria, Usuários, Configurações.

## O que já existe (base atual)

- Login + 4 papéis (admin, validador, operador, espectador).
- Mapa: 7 ruas × 84 vagas = 588 posições; status livre / bloqueado.
- Ações: dar entrada, dar saída, transferir, liberar múltiplas.
- Histórico (últimos 500) + gestão de usuários.
- Scanner de chassi (parcial), PWA instalável, Firebase Auth + Firestore.

## Status (modelo real do fluxo)

`livre` · `em estoque` (ocupada, sem reserva) · `reservada`
→ Expedido = sai para o cliente e libera a vaga (não é status, é evento).
Fluxo: **Em estoque → Reservado → [faturamento, fora do app] → Expedido**.

---

## PENDÊNCIAS IMEDIATAS (topo da fila)
1. ✅ **Regras do Firestore publicadas** (2026-07-20) — `hasProfile()` + `pedidos` no ar; mapa OK.
2. **Power Automate**: fluxo Excel/SharePoint → Firestore `pedidos` (auth por service account).
3. **Testar com o leitor Bluetooth** físico na estação de expedição.

---

## Fases

### Fase 0 — Fundação  ✅
- [x] Mapa carregando (588 vagas clicáveis).
- [x] Regras do Firestore publicadas (hasProfile + pedidos).

### Fase 1 — Dashboard & KPIs  ✅
- [x] Aba Dashboard: KPIs, taxa de ocupação, situação das vagas, por modelo e cor.
- [x] Status estendidos + cores no mapa (reservada âmbar).
- [ ] Rodapé: última sincronização / status. (não feito)

### Fase 2 — Ficha do Veículo  ⏳ (parcial)
- [x] Painel com modelo, cor, chassi, status, pedido/concessionária.
- [ ] Foto do veículo.
- [ ] Timeline de movimentações do veículo (histórico filtrado por chassi).
- [ ] Localização hierárquica Galpão/Rua/Baia/Posição.

### Fase 3 — Movimentações avançadas  ✅ (essencial)
- [x] Reservar (auto via pedido) e Expedir (dá baixa, com confirmação).
- [x] Leitura por chassi (leitor Bluetooth/USB, câmera ou digitação) na aba Expedição.
- [ ] Etiqueta imprimível por vaga (QR). (não feito)

### Fase 4 — Pedidos & Concessionárias  ⏳ (app pronto, falta o sync)
**Fonte da verdade:** Excel de pedidos no SharePoint (cliente, nº pedido, modelo, cor,
concessionária; chassi/motor preenchidos pela expedição). **Sync:** Power Automate empurra
para o Firestore quando a linha muda. **Chave de vínculo:** `chassi` (exato).

Fluxo: entrada no app (vaga ocupada c/ chassi) → expedição preenche chassi no Excel →
Power Automate → `pedidos` no Firestore → app casa `pedido.chassi === vaga.item.chassi`
→ marca **Reservado** e mostra nº pedido + concessionária.

- [x] App: listener em `pedidos` + reconciliação chassi→vaga (idempotente), auto-reserva.
- [x] Exibir pedido/cliente/concessionária no painel da vaga.
- [x] Aba **Expedição**: fila de reservados + leitura + baixa com confirmação.
- [ ] **Power Automate**: fluxo SharePoint Excel → Firestore REST (auth via service account).
- [ ] Cadastro/gestão de concessionárias.
- Decisões do Excel: nº pedido único, 1 veículo por linha, exibir nome da concessionária.

### Fase 5 — Relatórios, Auditoria & Alertas  (não iniciada)
- [ ] Relatórios (ocupação, giro, expedições) e exportação.
- [ ] Auditoria (quem fez o quê, quando) — base já existe no histórico.
- [ ] Alertas: veículos parados há X dias, divergências.

### Segurança
- [x] Regras endurecidas (`hasProfile()`), escape de e-mail no cabeçalho.
- [x] Regras publicadas no Console (2026-07-20).
- [ ] (Futuro) Mover criação de usuários para Cloud Function e desligar signup público.

---

## Decisões em aberto
- Manter arquivo único (`index.html`) ou modularizar? (hoje é single-file)
- Onde guardar fotos dos veículos? (Firebase Storage vs URL externa)
- Renomear repo/app para SRMV agora ou ao final?
