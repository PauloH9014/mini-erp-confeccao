# Domínio: Cadeia Produtiva de Confecção

## Problema
Uma confecção planeja uma quantidade exata de peças, mas o processo gera perdas:
material consumido a mais no corte e peças com defeito que viram produto de segunda.
Se 4 de 10 camisas saem com defeito, só 6 estão disponíveis, e se o consumo de
malha foi calculado errado, falta material para terminar a OP. Quando o sistema
continua informando 10, o e-commerce vende peças que não existem. O sistema registra
as perdas como movimentações de estoque, recalcula a necessidade de material pela
ficha técnica e informa só o estoque realmente disponível.

## Glossário
- **Referência**: identifica um modelo de peça e dá origem às suas variações.
- **Cor**: variação de uma referência.
- **Tamanho**: variação de uma referência.
- **Grade**: conjunto de tamanhos em que a referência é produzida e, na OP,
  a quantidade de cada tamanho.
- **Ficha técnica (BOM)**: o padrão seguido na produção, com os insumos e
  o consumo de cada um por tamanho. A BOM é a lista de materiais, e a ficha
  técnica também inclui o roteiro e as especificações, mas muitas empresas
  usam os dois nomes para a mesma coisa.
- **Ordem de produção (OP)**: calcula, a partir da grade, a matéria-prima
  necessária e o tempo estimado para concluir a produção.
- **Facção**: terceirizado que presta serviço de mão de obra, como a costura.
  O material enviado fica em estoque de terceiros, e a perda é apurada quando
  o lote retorna.
- **Movimentação de estoque**: registro de uma entrada, saída ou transferência
  de insumo ou produto, como a entrada de 10 camisas prontas ou a transferência
  de 4 para segunda.
- **Peça de segunda**: peça com defeito leve, vendida à parte do produto principal.
- **Rendimento**: quantos metros de tecido cabem em 1 kg de malha. Depende da
  gramatura e da largura, e permite converter consumo em metros para kg.
- **Coleção**: conjunto de referências lançadas juntas para uma estação ou período.

## Fluxo produtivo

```mermaid
flowchart LR
    A[Fio] --> B[Malha] --> C[Tinturaria] --> D[Corte]
    D --> E[Costura / Facção] --> F[Acabamento] --> G[Expedição]
```

- **Fio e malha**: o fio é transformado em tecido de malha, que entra no
  estoque como insumo, medido em kg.
- **Tinturaria**: a malha é tingida e ganha a cor. O estoque troca de malha
  crua para malha tingida, e pode haver perda de peso no processo.
- **Corte**: os cortes seguem a ficha técnica e a grade. Sai malha do estoque,
  e é aqui que começam as perdas de material.
- **Costura / Facção**: a peça cortada é montada na própria fábrica ou enviada
  a uma facção. Na facção, o material fica em estoque de terceiros.
- **Acabamento**: as peças são checadas e finalizadas. É onde muitas viram
  peça de segunda, e as boas entram como produto acabado.
- **Expedição**: o produto é enviado ao e-commerce ou direto ao cliente. É aqui
  que a contagem do estoque precisa estar correta.

## Decisões de modelagem que vêm do domínio
- **Por que o consumo de material muda por tamanho:** cada tamanho tem medidas
  diferentes, então uma camisa GG gasta mais tecido que uma P ou M. Se o sistema
  usasse um consumo único, o cálculo de kg da OP ficaria errado: sobraria material
  nos tamanhos pequenos e faltaria nos grandes. Por isso a ficha técnica guarda o
  consumo por tamanho e a OP soma conforme a quantidade de cada tamanho na grade.
- **Por que a ficha técnica é versionada:** a ficha diz quanto de tecido cada
  tamanho consome. Se ela for editada com uma OP já em produção, a OP passa a
  mostrar um consumo diferente do que foi usado, e o cálculo de material e os
  outros processos ficam errados. Por isso cada alteração gera uma nova versão,
  e a OP guarda a versão com que foi criada.
- **Por que o estoque é um ledger de movimentações:** uma OP de 10 camisas
  gera 10 peças, mas 4 saem com defeito e viram segunda. Se o sistema guardasse
  só o saldo, ele passaria de 10 para 6 e ninguém descobriria depois por que caiu
  nem para onde as 4 foram. Por isso o sistema registra cada movimentação (entrada
  de 10, transferência de 4 para segunda) e calcula o saldo somando todas elas.
  Assim qualquer diferença tem explicação, e um erro se corrige com uma nova
  movimentação, sem apagar o histórico.

## Perguntas em aberto
- Como o sistema registra o retorno de um lote da facção (peças aprovadas,
  em retrabalho, com perda definitiva e não localizadas)?
- O sistema vai modelar a tinturaria como etapa própria ou tratar a malha
  tingida como um insumo comprado?
- O sistema vai modelar o enfesto, que aparece no fluxo do ERP da RZ Sistemas,
  ou tratar o corte como uma etapa única?
- Como o sistema vai guardar a unidade de consumo e converter entre metros e kg?

## Hipóteses a validar
Coisas que assumi sobre o negócio e o que as fontes consultadas confirmaram.

| Hipótese | Fonte | Status |
|---|---|---|
| A perda da facção é apurada na conferência do retorno do lote, por lote e tamanho. Nem toda diferença é perda definitiva: parte pode estar em retrabalho ou ainda não localizada | Ref. 1 | Validado |
| A ficha técnica guarda uma unidade para cada insumo (kg, metro, peça), e unidade diferente da do estoque é um erro comum | Ref. 3 | Validado |
| A tinturaria acontece entre a malha e o corte e pode ser feita pela própria confecção ou por terceiros | Nenhuma ainda | A validar |
| Tecido plano costuma ser comprado por metro e malha por kg, e o consumo em metros é convertido para kg pelo rendimento | Nenhuma ainda | A validar |
| Existe um limite de perda tolerado na facção, combinado com a confecção | Nenhuma ainda | A validar |
| As peças com defeito leve são identificadas na conferência e viram peça de segunda, ou passam antes por retrabalho | Ref. 1 cita o retrabalho, não a peça de segunda | A validar |

## Referências
Os três são blogs de empresas que vendem ERP, úteis para entender o domínio.

1. ConfecçãoPro. [Como controlar facções de costura e evitar peças perdidas](https://confeccaopro.com.br/blog/tecnologia-na-confeccao/como-controlar-faccoes-de-costura-e-evitar-pecas-perdidas-com-um-sistema-de-controle-de-producao)
2. RZ Sistemas. [Sistema ERP para malharia: o que é e como funciona](https://www.sistemaparaconfeccao.com.br/sistema-erp-para-malharia-oque-e-como-funciona/)
3. UmPontoDois. [Ficha técnica e estrutura de produto (BOM)](https://blog.umpontodois.com.br/ficha-tecnica-estrutura-de-produto/)








