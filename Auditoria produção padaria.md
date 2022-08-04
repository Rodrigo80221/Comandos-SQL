```sql

select t1.CD_PRODUTO,t1.CustoHistorico,t1.CustoFormulasVigencia,t2.CustoProducaoPadaria  from
(

select f.CD_PRODUTO, sum(c.CUSTOGERENCIAL * f.QT_ITEM) / ( pp.QtdProduzida - pp.Perda) [CustoFormulasVigencia] , c2.CUSTOGERENCIAL [CustoHistorico]
from FN_Formulas(getdate()) f
inner join ProducaoPadaria pp on f.CD_PRODUTO = pp.CodProduto 
inner join VW_ProdutoCustoLoja c on f.CD_ITEM = c.CODPRODUTO  and c.CODLOJA = 1
LEFT join VW_ProdutoCustoLoja c2 on f.CD_PRODUTO = c2.CODPRODUTO  and c2.CODLOJA = 1
GROUP BY f.CD_PRODUTO,c2.CUSTOGERENCIAL ,pp.QtdProduzida, pp.Perda
) as t1

inner join

(

select pp.CodProduto, sum(c.CUSTOGERENCIAL * (ppp.QtdUtilizada / p.QtdRefUndBasica)) / ( pp.QtdProduzida - pp.Perda) [CustoProducaoPadaria] , c2.CUSTOGERENCIAL [CUSTO HISTÓRICO]  
from ProducaoPadaria pp 
inner join ProducaoPadariaProdutos ppp on pp.CodProduto = ppp.CodProduto 
inner join produtos p on p.codigo = ppp.CodItem
left join VW_ProdutoCustoLoja c on ppp.CodItem = c.CODPRODUTO  and c.CODLOJA = 1
LEFT join VW_ProdutoCustoLoja c2 on pp.CodProduto = c2.CODPRODUTO  and c2.CODLOJA = 1
GROUP BY pp.CodProduto,c2.CUSTOGERENCIAL ,pp.QtdProduzida, pp.Perda

) as t2 on t1.CD_PRODUTO = t2.CodProduto 

```
