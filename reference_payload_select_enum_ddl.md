---
name: reference_payload_select_enum_ddl
description: "GOTCHA de DDL em prod — campo select do Payload vira pgEnum, o DDL manual precisa CREATE TYPE + coluna enum, senão diverge do que o Drizzle espera"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-08-10T21:00:28.079Z
---

Quando uma coleção nova da área de membros tem um campo `type:'select'`, o push do Payload (ligado só no banco de TESTE) cria a coluna como um **pgEnum** (`enum_<colecao>_<campo>`), e se o campo é `required:true` ela vira **NOT NULL** — NÃO é `varchar`. Como prod tem push OFF (DDL manual, ver [[reference_neon_bancos]]), o `.sql` precisa espelhar isso: `DO $$ BEGIN CREATE TYPE enum_... AS ENUM (...); EXCEPTION WHEN duplicate_object THEN null; END $$;` + a coluna declarada com esse enum. Escrever `varchar` diverge do schema que o Drizzle espera e é a origem clássica do "erro de enum inválido depois do deploy".

**Como reconciliar sempre:** depois do push no banco de teste, inspecionar o schema real (`information_schema.columns.udt_name`, `pg_enum`, `pg_indexes`) e copiar tipos + índices pro DDL de prod — inclui os índices standalone que `index:true` gera e os de `created_at`/`updated_at` que o Payload cria por padrão. Peguei isso na Fase 2a (coleção `publicacoes-sociais`, campo `tipo` post/reel/story) só no review final; o implementer tinha escrito `tipo varchar`. Vale pra qualquer coleção nova com select. Relacionado: [[reference_suite_testes_area_membros]].
