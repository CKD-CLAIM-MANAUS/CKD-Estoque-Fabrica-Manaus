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

## Status estendidos (novo modelo)

`livre` · `ocupada` · `reservada` · `em expedição` · `bloqueada`
→ Dashboard: totais, livres, ocupadas, reservadas, expedição, bloqueadas, **taxa de ocupação**.

---

## Fases

### Fase 0 — Fundação (pré-requisito) ⏳
- [ ] Publicar regras do Firestore (`firestore.rules`) + desativar signup público.
- [ ] Confirmar mapa carregando (588 vagas clicáveis).
- [ ] Semear documento `estoque/vagas` inicial se necessário.

### Fase 1 — Dashboard & KPIs
- [ ] Nova aba Dashboard com cartões de indicadores e taxa de ocupação.
- [ ] Status estendidos (reservada, expedição) no modelo de dados e no mapa (cores).
- [ ] Rodapé: última sincronização / status.

### Fase 2 — Ficha do Veículo
- [ ] Painel de detalhe com VIN/chassi, modelo, cor, foto, status colorido.
- [ ] Localização completa (Galpão/Rua/Baia/Posição).
- [ ] Timeline de movimentações do veículo.

### Fase 3 — Movimentações avançadas
- [ ] Ações Reservar e Expedir (além de entrada/saída/transferir).
- [ ] Movimentação rápida por leitura de VIN / QR code.
- [ ] Etiqueta imprimível por vaga (QR).

### Fase 4 — Pedidos & Concessionárias  ← definida
**Fonte da verdade:** Excel de pedidos no SharePoint (cliente, nº pedido, modelo, cor,
concessionária; chassi/motor preenchidos pela expedição). **Sync:** Power Automate empurra
para o Firestore quando a linha muda. **Chave de vínculo:** `chassi` (exato).

Fluxo: entrada no app (vaga ocupada c/ chassi) → expedição preenche chassi no Excel →
Power Automate → `pedidos` no Firestore → app casa `pedido.chassi === vaga.item.chassi`
→ marca **Reservado** e mostra nº pedido + concessionária.

- [ ] Coleção `pedidos` no Firestore (schema definido a partir do Excel real).
- [ ] App: listener em `pedidos` + reconciliação chassi→vaga (idempotente), auto-reserva.
- [ ] Exibir pedido/cliente/concessionária no painel da vaga e no mapa.
- [ ] Power Automate: fluxo SharePoint Excel → Firestore REST (auth via service account).
- [ ] (Depois) Cadastro/gestão de concessionárias.

### Fase 5 — Relatórios, Auditoria & Alertas
- [ ] Relatórios (ocupação, giro, expedições) e exportação.
- [ ] Auditoria (quem fez o quê, quando).
- [ ] Alertas: veículos parados há X dias, divergências.

---

## Decisões em aberto
- Manter arquivo único (`index.html`) ou modularizar? (hoje é single-file)
- Onde guardar fotos dos veículos? (Firebase Storage vs URL externa)
- Renomear repo/app para SRMV agora ou ao final?
