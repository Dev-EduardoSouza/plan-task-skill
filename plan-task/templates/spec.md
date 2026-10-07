# Spec <nn> — <título imperativo, vira mensagem de commit>

**Task:** [<ID>](../<ID>.md) · **Status:** ⬜ Pendente

## O que será feito

<peças concretas, com nome real. Ex.: "Migration `V31__CriarTabelaExemplo` e model `TabelaExemplo`. A tabela terá os campos:">

```
id                   BIGINT PK auto
desc_tabela_exemplo  VARCHAR(120) NOT NULL
data_criacao         DATETIME NOT NULL
user_id              BIGINT NOT NULL
```

<Estrutura já mostrada em outra spec é referenciada ("consome o payload da spec 02"), não copiada.>

**Arquivos:** `<arquivo>`, `<arquivo novo>`
**Pronto quando:** <comportamento observável, em linguagem de produto>

---

# Execução

## Critérios

- [ ] <resultado esperado campo a campo> — Validar: `<comando>`
- [ ] <caminho de erro: status e corpo> — Validar: `<comando>`
- [ ] Teste de <regra> — Validar: `<comando>`

**Feito:** <solução final, 1-3 linhas>
**Observado:** <payload/saída real resumida>
**Não funcionou:** <`abordagem → sintoma`, uma por linha, ou vazio>
**Commit:** <hash + assunto>
