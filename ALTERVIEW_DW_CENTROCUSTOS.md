```sql
ALTER VIEW [dbo].[DW_BI_LancamentosFinanceirosCentrosCustos] AS 
SELECT CodLancamentoFinanceiro, CodLojaCentrosCusto, CodCentroCusto, Percentual FROM GESTAO_GIRARDI.dbo.LancamentosFinanceirosCentrosCusto 


union all


-- Aqui foram gerados os centros de custos para os juros dos lançamentos financeiros a pagar 

SELECT  
LF.CodLancamentoFinanceiro * - 1 CodLancamentoFinanceiro,
CC.Codlojacentroscusto Codlojacentroscusto,
CC.CodCentroCusto CodCentroCusto,
CC.Percentual Percentual

FROM LancamentosFinanceiros LF 
inner join CentroCustoPlanoContas cc on lf.CodLojaLancamento = cc.codlojacentroscusto
and codplanoconta = (SELECT CodConta FROM PLANOCONTAS WHERE CODESTRUTURAL = (SELECT VALOR FROM Configuracoes WHERE Descricao LIKE 'ContaJurosPagos'))

WHERE  
year(convert(date, LF.DataVencimento)) in (year(getdate())-2,year(getdate())-1,year(getdate()))  
AND  
LF.CodContaDebito in ( 
                 SELECT PC.CodConta  
                 from PlanoContas PC  
                 WHERE  
                 PC.EstruturaContasPagar = 1  -- AQUI VAMOS PEGAR OS LANÇAMENTOS A RECEBER
                 ) 
and LF.Juros > 0


-- union all taxas dos lançamentos a pagar 
-- union all multas dos lançamentos a pagar 
-- union all descontos dos lançamentos a pagar 

-- union all taxas dos lançamentos a receber 
-- union all multas dos lançamentos a receber 
-- union all juros dos lançamentos a receber 
-- union all descontos dos lançamentos a receber 

```
