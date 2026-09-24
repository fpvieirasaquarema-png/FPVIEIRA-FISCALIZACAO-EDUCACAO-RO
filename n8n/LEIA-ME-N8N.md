# Fluxo n8n — notifica O.S. da Fiscalização (e-mail COM ANEXO + WhatsApp no grupo)

O que ele faz, sozinho, a cada 2 minutos:
1. Busca no Supabase da Fiscalização as O.S. novas (ainda não notificadas);
2. Baixa a planilha oficial do Storage (a mesma do link, com timbre);
3. **Envia o e-mail no padrão conservando**: assunto `OS <nº>`, corpo-resumo e
   o **xlsx ANEXADO**, para fp.edu.riodasostras@gmail.com;
4. **Posta o arquivo no grupo da equipe do fiscal** via Evolution API (chega
   para todo mundo — WhatsApp normal e Business — sem ninguém compartilhar
   nada). A legenda leva o **link do app de campo**: o colaborador clica e cai
   na Nova O.S. já preenchida (v109 do FPV-Campo), só registra a execução;
5. Marca a O.S. como notificada (nunca duplica).

## Antes de importar — 1 SQL (SQL Editor do projeto da FISCALIZAÇÃO)

```sql
alter table os_fiscalizacao add column if not exists notificado_em timestamptz;
-- as O.S. antigas (2401-2408) já foram divulgadas manualmente — marca p/ o robô não repetir:
update os_fiscalizacao set notificado_em = now() where numero <= 2408 and notificado_em is null;
select 'coluna ok' as resultado;
```

(Se quiser que o robô REENVIE alguma delas no padrão novo, é só não marcar
essa O.S. — tira o número do `update` acima.)

## Importar e ligar (5 minutos)

1. n8n → **Workflows → Import from file** → `notifica-os-fiscais.json`
   (workflow NOVO — não colar dentro de um canvas existente, senão duplica nós).
2. Trocar os 2 SEGREDOS que sobraram (Ctrl+F por `COLE_AQUI`):
   - `COLE_AQUI_A_SERVICE_KEY` (nós Busca e Marca) → service key do Supabase da
     **Fiscalização** (Settings → API). Ela fica SÓ no n8n (VPS), nunca no app.
     ⚠ NÃO confundir com a anon — as duas começam com `eyJ` (erro real de 10/09).
   - `COLE_AQUI_EVOLUTION_APIKEY` (nó WhatsApp) → a apikey da instância Renanfp.
   O resto JÁ ESTÁ PREENCHIDO no arquivo (24/09): URL do envio
   `https://evo.editly.com.br/message/sendMedia/Renanfp` e os 3 JIDs verificados
   em 11/09 (grupo do Renato, do Wellington e o padrão "RDO - Manutenção
   Preventiva/Emergencial") — não colar nada por cima deles.
3. No nó **"E-mail oficial COM ANEXO"**: selecionar a credencial Gmail já
   cadastrada no n8n (a "FPV"/Click que os outros fluxos usam).
4. Ativar o workflow. Teste: emitir uma O.S. no app → em até 2 min chega o
   e-mail com anexo E o arquivo no grupo.

## Notas

- O e-mail sai da conta do n8n (Click/conservando) — **padrão idêntico** ao
  das O.S. da SEMDE: assunto `OS <nº>` + xlsx anexado. O robô da Educação
  (fluxo antigo) vai processá-lo como qualquer O.S. — sem ajuste no filtro.
- Os botões WhatsApp/E-mail do app continuam funcionando (com link) — viram
  reforço manual; a divulgação oficial passa a ser deste robô.
- Se a Evolution recusar `media` por URL, trocar o nó do WhatsApp para enviar
  o binário (`$binary.data`) — me chame que ajusto o nó.
