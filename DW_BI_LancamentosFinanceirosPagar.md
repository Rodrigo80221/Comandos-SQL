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


--//lançamentos financeiros referentes a estorno nas notas de saída do tipo de operação Estorno de NF-e (DÉBITO) 

SELECT  
CodNotaSaida * - 1, 
0, --LF.CodContaCredito as CodContaMovimentacao, ???????????????
LF2.CodContaDebito as CodContaPagar, 
 (select top 1 descricao from GESTAO_GIRARDI.dbo.FormasPagamentoFinanceiro), --(select descricao from GESTAO_GIRARDI.dbo.FormasPagamentoFinanceiro where codformapagamento = LF.CodFormaPagamento) as FormaPagamento, 
'QUITADO', -- case LF.CodSituacao when 0 then 'VENCIDO' when 1 then 'EM ABERTO' when 2 then 'QUITADO' when 3 then 'CANCELADO' when 4 then 'CHEQUE EM ABERTO' end as Situacao,  
null, --LF.CodLancamentoParcela1, 
0, --LF.Cancelado, 
LF2.CodLoja, --LF.CodLojaLancamento, 
(select top 1 codigo from GESTAO_GIRARDI.dbo.Operador where OperadorTelecon = 1 order by codigo ), --  LF.CodOperadorRegistro, 
LF2.DT_EMISSAO, --LF.DataHoraRegistro, 
LF2.DT_EMISSAO, --LF.DataVencimento, 
0, --LF.Desconto, 
'lançamentos financeiros referentes a estorno NF_Saidas', --LF.Historico, 
0, --LF.HouveAlteracaoManual, 
0, --LF.Juros, 
0, --LF.Multa, 
LF2.NUMERO, --LF.NumeroDocumento, 
'', --LF.NumeroDocumentoOrigem, 
0, --LF.NumeroParcelas, 
'lançamentos financeiros referentes a estorno NF_Saidas', --LF.ObservacaoFinanceira, 
'1/1', --LF.Parcela, 
0, --LF.Taxa, 
'BaixaAutomatica', --case when LF.TipoBaixaAutomatica = 0 then 'BaixaManual' else 'BaixaAutomatica' end as TipoBaixa,  
LF2.TOTAL, --LF.Valor, 
LF2.CD_CLIENTE, --LF.CodFornecedor, 
LF2.CodLoja, --LF.CodLojaPagamento, 
NULL, --LF.CodNotaEntrada, 
NULL, --LF.CodOperadorPagamento, 
LF2.DT_EMISSAO, --LF.DataHoraPagamento, 
NULL, --LF.CodFaturaOrigem, 
LF2.DT_EMISSAO --LF.DataCompetencia 
FROM 
(
    SELECT TP.Conta as CodContaDebito, DEB.CodEstrutural as CodEstruturalDebito , 
    SUM(NS.TotalProdutosServicos) AS TOTAL,   NS.CD_NOTA as CodNotaSaida, NS.DT_EMISSAO as DT_EMISSAO, NS.NUMERO as NUMERO, NS.CD_CLIENTE,	NS.CodLoja as CodLoja 
    FROM GESTAO_GIRARDI.dbo.NF_Saidas NS WITH(NOLOCK) inner join GESTAO_GIRARDI.dbo.Tipos_Operacao TP WITH(NOLOCK) on NS.CodTipoOperacao = TP.Codigo 
    INNER JOIN GESTAO_GIRARDI.dbo.PlanoContas DEB WITH(NOLOCK) ON TP.Conta = DEB.CodConta 
    where NS.ChaveAcesso not in (select NFP.ChaveAcesso from GESTAO_GIRARDI.dbo.Nf_Entradas NS WITH(NOLOCK) inner join GESTAO_GIRARDI.dbo.NF_ENTRADA_PRODUTOR NFP WITH(NOLOCK) on NS.Cd_Nota = NFP.CodNotaEntrada
    where 
	CodTipoOperacao =  (select valor from GESTAO_GIRARDI.dbo.configuracoes where descricao = 'TIPO_OPERACAO_NFE_AJUSTE') -- CONFIGURAÇÃO FIXA
	and NS.Situacao_NF = 0 and NS.CodStatus = 2 and NS.Numero <> 0)
    and TP.UtilizaContaDRE = 1 and NS.Situacao_NF = 0 and NS.CodStatus = 2 and NS.Numero <> 0  
    AND    ( 
        EstruturaContasPagar = 1  -- NÃO BUSCAR MAIS PELO CÓDIGO ESTRUTURAL
      ) 
	And year(convert(date, NS.DT_EMISSAO)) in (year(getdate())-2,year(getdate())-1,year(getdate())) 
    GROUP BY TP.Conta, DEB.CodEstrutural, NS.CD_NOTA , NS.DT_EMISSAO, NS.NUMERO, NS.CD_CLIENTE, NS.CodLoja 
) as LF2 
WHERE ISNULL((SELECT VALOR FROM GESTAO_GIRARDI.dbo.CONFIGURACOES WHERE DESCRICAO = 'ADICIONAR_ESTORNOS_GESTAO_RELATORIOS'),1) = 1  -- CONFIGURAÇÃO PARA CANCELAMENTO


``` 
