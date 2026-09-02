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

- **Notas Fiscais (Entradas)** — lançamento das compras do período (uma linha por NF: data,
  nº da NF, fornecedor e valor total da nota).
- **Estoque Inicial** — inventário de materiais no início do período. O campo "Item" tem busca
  com autocompletar (baseada em `Itens estoque.xlsx`); ao escolher um item da lista, o "Valor
  Unit. (R$)" é preenchido automaticamente (mas continua editável, caso o preço tenha mudado).
- **Estoque Final** — mesma dinâmica da aba Estoque Inicial (busca de item + preço automático).
- **Resultado CMV** — cupom com o cálculo consolidado e campo opcional de "Vendas do período"
  para calcular % de CMV sobre vendas e margem bruta.

Cada tabela tem botões para **adicionar linha**, **limpar tabela** e **exportar CSV**.

## Período de referência

O campo "Período de referência" no topo filtra o que aparece em todas as abas: Notas Fiscais,
Estoque Inicial, Estoque Final e o Resultado (incluindo "Vendas do período") mostram **somente
os lançamentos daquele mês**. Trocar o período não apaga nada — os dados de outros meses
continuam guardados e reaparecem ao selecioná-los de novo. "Limpar tabela" e "Exportar CSV"
também agem apenas sobre o mês selecionado.

A seleção de período é local a cada pessoa (não é sincronizada) — cada viewer pode navegar por
um mês diferente sem afetar o que os outros estão vendo. Os dados de cada mês continuam
compartilhados normalmente entre todos.

## Como os dados são salvos

O app não usa banco de dados próprio: cada aba (Entradas / Estoque Inicial / Estoque Final /
Meta) é lida e gravada via `fetch` num **Google Apps Script Web App** já publicado (URL em
`API_URL`, dentro do `<script>` no topo do arquivo), que por sua vez lê/escreve numa planilha
Google. Isso é o que permite que **várias pessoas abram o mesmo `index.html` e vejam/editem os
mesmos dados**, como se fosse compartilhado por link:

- Ao editar qualquer campo, o app salva automaticamente ~0,6s depois (debounce). O botão
  **"Salvar agora"** no rodapé força o salvamento imediato e mostra se há algo pendente
  ("💾 Salvar agora") ou se está tudo gravado ("✓ Tudo salvo"). Se você tentar fechar a página
  com algo pendente, o navegador avisa antes de sair.
- **Trava de segurança:** cada aba só pode ser salva depois de ter sido lida com sucesso do
  servidor. Se a leitura falhar (queda de conexão, erro momentâneo do Apps Script), aparece um
  aviso vermelho e o salvamento fica **bloqueado**. Isso existe porque o salvamento reescreve a
  aba inteira: sem a trava, uma falha de leitura deixava o app com a lista vazia e o próximo
  salvamento apagava todos os lançamentos já gravados.
- **Backup automático:** antes de cada gravação, o Apps Script copia o conteúdo anterior da aba
  para a aba `_Backup` da planilha (com data/hora e a linha em JSON, mantendo as últimas ~5000
  linhas). Serve como rede de segurança para recuperar dados sobrescritos por engano.
- A cada 20s, se a aba do navegador estiver visível, o app busca atualizações do servidor —
  mas nunca recarrega uma aba que ainda tem alterações não confirmadas pelo servidor, para
  não sobrescrever o que a pessoa acabou de digitar com uma leitura desatualizada.
- Se a pessoa trocar de aba/app logo depois de digitar, o app força o salvamento na hora em
  vez de esperar o debounce.
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

## Catálogo de itens e preços

Os itens e valores unitários sugeridos nas abas de estoque vêm de `Itens estoque.xlsx` e estão
embutidos no `index.html` (constante `ITENS_ESTOQUE`, no `<script>`). Para atualizar preços ou
adicionar/remover itens, edite essa planilha e depois atualize a constante correspondente no
`index.html` (não há leitura automática do arquivo — ele fica no projeto só como fonte de
referência).

## Histórico

Este projeto consolida versões anteriores geradas em 31/08/2026 que estavam espalhadas em
`Downloads` (`sistema-cmv-pizzahut*.html`, `cmv-pizzahut-standalone*.html`) e uma cópia com
bug de sintaxe (URL do Apps Script sem aspas) salva por engano como `CMV.txt` dentro da pasta
de documentos `3. FCA_PIZZA HUT\2026`. A versão aqui é a mais completa (última gerada),
corrigida com a URL real do Apps Script.
