```sql
--SELECT PARA COMPARAR O ÚLTIMO CUSTO DO PRODUTO COM O CUSTO ATUAL CALCULADO PELAS FÓRMULAS VIGÊNCIA COM O CUSTO ATUAL CALCULADO PELA TELA DE PRODUÇÃO PADARIA

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


-- SELECT PARA VERIFICAR SE O TOTAL DE ITENS NAS FORMULAS VIGENCIA SÃO IGUAIS AOS TOTAIS DE ITENS CADASTRADOS NA PRODUÇÃO PADARIA 
select t1.CD_PRODUTO,t1.totalItens,t2.totalItens from
(
SELECT CD_PRODUTO , count(cd_Item) totalItens FROM FN_Formulas(getdate())
group by CD_PRODUTO 
) as t1 
inner join 
(
select codproduto, count(coditem) totalItens from ProducaoPadariaProdutos 
group by codproduto
) as t2 on t1.CD_PRODUTO = t2.CodProduto
where t1.totalItens <> t2.totalItens 


-- SELECT PARA COMPARAR O QTD DAS FORMULAS VIGENCIA COM O CALCULO DOS VOLUMES DOS PRODUTOS NA PRODUÇÃO PADARIA 

select fv.CodProduto , fv.CodItem , fv.QtItem ,CONVERT(float,(ppp.QtdUtilizada / p.QtdRefUndBasica)) QtItemProducaoPadaria , T1.maxData
from FormulasVigencia fv inner join
(
select fv.CodProduto , max(fv.DtFormula) maxData 
from FormulasVigencia fv 
where fv.CodProduto in (select distinct CodProduto from ProducaoPadariaProdutos )
group by fv.CodProduto 
) t1 on fv.CodProduto = t1.CodProduto and fv.DtFormula = t1.maxData 
inner join ProducaoPadariaProdutos ppp on fv.CodProduto = ppp.CodProduto and fv.CodItem = ppp.CodItem 
inner join Produtos P on fv.CodItem = p.Codigo 

-- CORRIGIR TABELA DE FÓRMULAS VIGENCIA COM O CÁLCULO DA PRODUÇÃO PADARIA 

UPDATE FV SET fv.QtItem = CONVERT(float,(ppp.QtdUtilizada / p.QtdRefUndBasica)) 
from FormulasVigencia fv inner join
(
select fv.CodProduto , max(fv.DtFormula) maxData 
from FormulasVigencia fv 
where fv.CodProduto in (select distinct CodProduto from ProducaoPadariaProdutos )
group by fv.CodProduto 
) t1 on fv.CodProduto = t1.CodProduto and fv.DtFormula = t1.maxData 
inner join ProducaoPadariaProdutos ppp on fv.CodProduto = ppp.CodProduto and fv.CodItem = ppp.CodItem 
inner join Produtos P on fv.CodItem = p.Codigo 
```
