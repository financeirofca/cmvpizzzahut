# Sistema de Cálculo de CMV — Pizza Hut

Aplicativo de página única (`index.html`) para apurar o **CMV (Custo da Mercadoria Vendida)**
mensal das lojas Pizza Hut, com os dados salvos numa planilha Google compartilhada.

## Fórmula

```
CMV = Estoque Inicial + Compras (Notas Fiscais) − Estoque Final
```

O resultado, o % de CMV sobre as vendas e a margem bruta são recalculados automaticamente
a cada alteração.

## Abas

- **Notas Fiscais (Entradas)** — lançamento das compras de insumos do período (uma linha por
  item de NF: data, nº da NF, fornecedor, item, categoria, quantidade, valor unitário).
- **Estoque Inicial** — inventário de materiais no início do período.
- **Estoque Final** — inventário de materiais no fim do período.
- **Resultado CMV** — cupom com o cálculo consolidado, campo opcional de "Vendas do período"
  para calcular % de CMV sobre vendas e margem bruta, e o custo médio de reposição.

Cada tabela tem botões para **adicionar linha**, **limpar tabela** e **exportar CSV**.

## Como os dados são salvos

O app não usa banco de dados próprio: cada aba (Entradas / Estoque Inicial / Estoque Final /
Meta) é lida e gravada via `fetch` num **Google Apps Script Web App** já publicado (URL em
`API_URL`, dentro do `<script>` no topo do arquivo), que por sua vez lê/escreve numa planilha
Google. Isso é o que permite que **várias pessoas abram o mesmo `index.html` e vejam/editem os
mesmos dados**, como se fosse compartilhado por link:

- Ao editar qualquer campo, o app salva automaticamente ~0,6s depois (debounce).
- A cada 20s, se a aba do navegador estiver visível, o app busca atualizações do servidor
  (sem sobrescrever uma tabela em que a pessoa esteja digitando no momento).
- O rodapé mostra o horário da última sincronização, ou um aviso se o salvamento falhar
  (verifique a conexão e se a URL do Apps Script ainda está publicada).

Se `API_URL` ficar vazia ou inválida, o app mostra um aviso vermelho e passa a funcionar só
localmente (sem salvar nem compartilhar).

## Como usar / compartilhar

Não há build nem instalação: é um único arquivo HTML.

- Para usar sozinho: dê duplo clique em `index.html` (abre no navegador padrão).
- Para compartilhar com a equipe: envie o arquivo `index.html` (por e-mail, Drive, Dropbox
  etc.) — todos que abrirem o arquivo e tiverem acesso à internet vão ler/gravar na mesma
  planilha Google por trás do Apps Script.

## Histórico

Este projeto consolida versões anteriores geradas em 31/08/2026 que estavam espalhadas em
`Downloads` (`sistema-cmv-pizzahut*.html`, `cmv-pizzahut-standalone*.html`) e uma cópia com
bug de sintaxe (URL do Apps Script sem aspas) salva por engano como `CMV.txt` dentro da pasta
de documentos `3. FCA_PIZZA HUT\2026`. A versão aqui é a mais completa (última gerada),
corrigida com a URL real do Apps Script.
