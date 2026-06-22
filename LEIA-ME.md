# Pedro 💙 — Gestação, Enxoval & Chá de Fraldas

App local pra acompanhar tudo do Pedro num lugar só, com **abas laterais**:

- **🏠 Início** — mostra, **atualizado automaticamente todo dia**, com quantas semanas e dias
  está a gestação e quantos dias faltam pro nascimento, além de um resumo do próximo exame, do
  enxoval e da contagem pra festa.
- **👶 Pedro** — nome completo, idade gestacional (você informa uma vez e a contagem anda
  sozinha) e os próximos exames com a obstetra (data e horário).
- **🧸 Enxoval** — checklist + controle financeiro (igual já era): sobe planilha, reconhece
  cabeçalhos, progresso, filtros, presentes/lojas e alertas.
- **🎉 Chá de fraldas** — mesma pegada do enxoval, pra organizar o que os convidados trazem **e**
  os investimentos da festa, com orçamento e **contagem regressiva até a data da festa**.

## Como usar (agora, offline)

1. Abra o arquivo **`index.html`** com 2 cliques (abre no navegador). Não precisa instalar nada.
2. Na aba **Pedro**, informe o nome e quantas semanas/dias está a gestação — a aba **Início**
   passa a contar sozinha (amanhã já mostra +1 dia, sem você mexer).
3. Nas abas **Enxoval** e **Chá de fraldas**, clique em **"Subir planilha"** (ou **"+ Item"**).
   Confira o **de–para** dos cabeçalhos (já vem preenchido — o app detecta sozinho) e importe.
4. Marque o que já foi, preencha valor pago e quem trouxe. **Tudo salva sozinho** no navegador.

> A contagem de gestação é calculada a partir da **data de referência** + semanas/dias que você
> informou. Se a conta da obstetra mudar, é só atualizar na aba Pedro.

> Dica: clique em **"Ver exemplo"** na primeira tela pra ver o app cheio antes de importar.

### Cabeçalhos reconhecidos automaticamente
`Categoria · Item · Qtd alvo · Preço estimado (un) · Subtotal estimado · Prioridade ·
Comprar quando · OK? · Qtd comprada · Valor real pago · Quem presenteou / loja`

Variações com/sem acento, maiúsculas e nomes parecidos também são reconhecidas. Se algo não
casar, é só ajustar na tela de mapeamento. O **Subtotal** é sempre **calculado** (qtd × preço),
então a coluna da planilha é ignorada de propósito.

## Backup (importante!)

Enquanto está só no navegador, os dados ficam **neste computador/navegador**. Para não perder
e para passar de um aparelho pro outro, use o menu **"Backup"** no topo:

- **Exportar backup (JSON)** → salva um arquivo com tudo. Guarde de vez em quando.
- **Importar backup (JSON)** → restaura a partir desse arquivo.
- **Exportar para CSV** → abre no Excel/Google Sheets quando quiser.

## Hospedar online depois (vocês dois, no celular)

O app já está pronto pra virar site. Há **dois níveis**:

### Nível 1 — Site online (cada um no seu aparelho, dados separados)
É só publicar a pasta. Não precisa programar:
- **Netlify Drop**: acesse <https://app.netlify.com/drop> e **arraste a pasta `enxoval-bebe`**.
  Pronto, vira um link `https://...`. (Vercel e GitHub Pages também servem.)
- Funciona no celular, mas cada aparelho guarda os dados localmente (use o backup JSON pra
  igualar os dois).

### Nível 2 — Sincronizado de verdade (mesma lista, tempo real, os dois juntos)
Para os dois verem a **mesma lista atualizada**, basta plugar um banco na nuvem. O app foi feito
pra isso: **toda a gravação passa por um único objeto `Store`** no começo do `index.html`:

```js
const Store = {
  async load(){ /* hoje: localStorage */ },
  async save(s){ /* hoje: localStorage */ }
};
```

Para sincronizar, troque **só essas duas funções** por chamadas a um backend — recomendo o
**Supabase** (plano grátis, em tempo real, mínimo de configuração):

1. Crie um projeto em <https://supabase.com> e uma tabela `enxoval` (uma linha = o estado, ou
   uma linha por item).
2. No `index.html`, em `Store.load()` faça um `fetch`/SDK do Supabase para **ler**, e em
   `Store.save()` para **gravar**. Nenhuma outra parte do app precisa mudar — a interface toda
   já conversa só com o `Store`.
3. (Opcional) ative *realtime* do Supabase para a lista atualizar na tela do outro na hora.

> Resumo: **nada de reescrever o app** — é um ajuste localizado numa única peça.

## Arquivos

- `index.html` — o app inteiro (interface + lógica), auto-contido.
- `vendor/xlsx.full.min.js` — biblioteca que lê planilhas Excel, **embutida** pra funcionar
  offline (sem depender de internet).
- `LEIA-ME.md` — este arquivo.

## Privacidade

100% local: enquanto não hospedar, **nada sai do seu navegador**. Sem cadastro, sem nuvem,
sem rastreio.
