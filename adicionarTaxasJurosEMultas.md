``` sql
ALTER VIEW [dbo].[DW_BI_LancamentosFinanceirosPagar] as 

SELECT  
LF.CodLancamentoFinanceiro, 
LF.CodContaCredito as CodContaMovimentacao, 
LF.CodContaDebito as CodContaPagar, 
(select descricao from GESTAO_GIRARDI.dbo.FormasPagamentoFinanceiro where codformapagamento = LF.CodFormaPagamento) as FormaPagamento, 
case LF.CodSituacao when 0 then 'VENCIDO' when 1 then 'EM ABERTO' when 2 then 'QUITADO' when 3 then 'CANCELADO' when 4 then 'CHEQUE EM ABERTO' end as Situacao,  
LF.CodLancamentoParcela1, 
LF.Cancelado, 
LF.CodLojaLancamento, 
LF.CodOperadorRegistro, 
LF.DataHoraRegistro, 
LF.DataVencimento, 
LF.Desconto, 
LF.Historico, 
LF.HouveAlteracaoManual, 
LF.Juros, 
LF.Multa, 
LF.NumeroDocumento, 
LF.NumeroDocumentoOrigem, 
LF.NumeroParcelas, 
LF.ObservacaoFinanceira, 
LF.Parcela, 
LF.Taxa, 
case when LF.TipoBaixaAutomatica = 0 then 'BaixaManual' else 'BaixaAutomatica' end as TipoBaixa,  
LF.Valor, 
LF.CodFornecedor, 
LF.CodLojaPagamento, 
LF.CodNotaEntrada, 
LF.CodOperadorPagamento, 
LF.DataHoraPagamento, 
LF.CodFaturaOrigem, 
LF.DataCompetencia 
FROM GESTAO_GIRARDI.dbo.LancamentosFinanceiros LF 
WHERE  
year(convert(date, LF.DataVencimento)) in (year(getdate())-2,year(getdate())-1,year(getdate()))  
AND  
LF.CodContaDebito in ( 
                 SELECT PC.CodConta  
                 FROM GESTAO_GIRARDI.dbo.PlanoContas PC  
                 WHERE  
                 PC.EstruturaContasPagar = 1  
                 ) 


union all

-- CRIA LANÇAMENTOS "FAKES" PARA JUROS , TAXAS E MULTAS. 

SELECT  
LF.CodLancamentoFinanceiro * - 1, 
LF.CodContaCredito as CodContaMovimentacao, 

-- AQUI COLOCAMOS A CONTA REFERENTE AOS JUROS PAGOS 
(SELECT CodConta FROM PLANOCONTAS WHERE CODESTRUTURAL = (SELECT VALOR FROM Configuracoes WHERE Descricao LIKE 'ContaJurosPagos')) as CodContaPagar,  --LF.CodContaDebito as CodContaPagar, 

(select descricao from GESTAO_GIRARDI.dbo.FormasPagamentoFinanceiro where codformapagamento = LF.CodFormaPagamento) as FormaPagamento, 
case LF.CodSituacao when 0 then 'VENCIDO' when 1 then 'EM ABERTO' when 2 then 'QUITADO' when 3 then 'CANCELADO' when 4 then 'CHEQUE EM ABERTO' end as Situacao,  
LF.CodLancamentoParcela1, 
LF.Cancelado, 
LF.CodLojaLancamento, 
LF.CodOperadorRegistro, 
LF.DataHoraRegistro, 
LF.DataVencimento, 
LF.Desconto, 
LF.Historico, 
LF.HouveAlteracaoManual, 
LF.Juros, 
LF.Multa, 
LF.NumeroDocumento, 
LF.NumeroDocumentoOrigem, 
LF.NumeroParcelas, 
LF.ObservacaoFinanceira, 
LF.Parcela, 
LF.Taxa, 
case when LF.TipoBaixaAutomatica = 0 then 'BaixaManual' else 'BaixaAutomatica' end as TipoBaixa,  
LF.Valor, 
LF.CodFornecedor, 
LF.CodLojaPagamento, 
LF.CodNotaEntrada, 
LF.CodOperadorPagamento, 
LF.DataHoraPagamento, 
LF.CodFaturaOrigem, 
LF.DataCompetencia 
FROM GESTAO_GIRARDI.dbo.LancamentosFinanceiros LF 
WHERE  
year(convert(date, LF.DataVencimento)) in (year(getdate())-2,year(getdate())-1,year(getdate()))  
AND  
LF.CodContaDebito in ( 
                 SELECT PC.CodConta  
                 FROM GESTAO_GIRARDI.dbo.PlanoContas PC  
                 WHERE  
                 PC.EstruturaContasPagar = 1  
                 ) 

and LF.Juros > 0

-- union all

-- Fazer um igual para as taxas 

-- union all

-- Fazer um igual para as multas


union all

-- CRIA LANÇAMENTOS "FAKES" DOS DESCONTOS QUE ESTÃO NAS RECEITAS 


SELECT  
LF.CodLancamentoFinanceiro * - 1, 
LF.CodContaCredito as CodContaMovimentacao, 

-- AQUI COLOCAMOS A CONTA REFERENTE AOS JUROS PAGOS 
(SELECT CodConta FROM PLANOCONTAS WHERE CODESTRUTURAL = (SELECT VALOR FROM Configuracoes WHERE Descricao LIKE 'ContaDescontosRecebidos')) as CodContaPagar,  --LF.CodContaDebito as CodContaPagar, 

(select descricao from GESTAO_GIRARDI.dbo.FormasPagamentoFinanceiro where codformapagamento = LF.CodFormaPagamento) as FormaPagamento, 
case LF.CodSituacao when 0 then 'VENCIDO' when 1 then 'EM ABERTO' when 2 then 'QUITADO' when 3 then 'CANCELADO' when 4 then 'CHEQUE EM ABERTO' end as Situacao,  
LF.CodLancamentoParcela1, 
LF.Cancelado, 
LF.CodLojaLancamento, 
LF.CodOperadorRegistro, 
LF.DataHoraRegistro, 
LF.DataVencimento, 
LF.Desconto, 
LF.Historico, 
LF.HouveAlteracaoManual, 
LF.Juros, 
LF.Multa, 
LF.NumeroDocumento, 
LF.NumeroDocumentoOrigem, 
LF.NumeroParcelas, 
LF.ObservacaoFinanceira, 
LF.Parcela, 
LF.Taxa, 
case when LF.TipoBaixaAutomatica = 0 then 'BaixaManual' else 'BaixaAutomatica' end as TipoBaixa,  
LF.Valor, 
LF.CodFornecedor, 
LF.CodLojaPagamento, 
LF.CodNotaEntrada, 
LF.CodOperadorPagamento, 
LF.DataHoraPagamento, 
LF.CodFaturaOrigem, 
LF.DataCompetencia 
FROM GESTAO_GIRARDI.dbo.LancamentosFinanceiros LF 
WHERE  
year(convert(date, LF.DataVencimento)) in (year(getdate())-2,year(getdate())-1,year(getdate()))  
AND  
LF.CodContaDebito in ( 
                 SELECT PC.CodConta  
                 FROM GESTAO_GIRARDI.dbo.PlanoContas PC  
                 WHERE  
                 PC.EstruturaContasReceber = 1  -- AQUI VAMOS PEGAR OS LANÇAMENTOS A RECEBER
                 ) 
and LF.Desconto > 0

```
