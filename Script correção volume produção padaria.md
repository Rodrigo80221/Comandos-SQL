```sql

/*

-- SELECT PARA VERIFICAR PRODUTOS QUE ESTÃO COM DIVERGÊNCIA (*** SE ESTÁ NA LOJA 1 ESTARÁ EM TODAS)

DECLARE @CodLoja AS INT SET @CodLoja = 1

SELECT CodProduto , [CustoGerencialProducaoPadaria], [CustoGerencialFormulasVigencia],  [CustoHistorico]  FROM
(
	select CodProduto,
	(case when CustoPorcentagem > 0 then [CustoGerencialProducaoPadaria] + (([CustoGerencialProducaoPadaria] * CustoPorcentagem) / 100) else [CustoGerencialProducaoPadaria] + CustoValor end) / QtdProduzidaFinal [CustoGerencialProducaoPadaria],
	(case when CustoPorcentagem > 0 then [CustoFormulasVigencia] + (([CustoFormulasVigencia] * CustoPorcentagem) / 100) else [CustoFormulasVigencia] + CustoValor end) / QtdProduzidaFinal [CustoGerencialFormulasVigencia],
	CustoHistorico 
	from 
	(
		select 

		pp.CodProduto,
		sum(c.CUSTOGERENCIAL * (ppp.QtdUtilizada / p.QtdRefUndBasica)) [CustoGerencialProducaoPadaria],
		pp.QtdProduzida - pp.Perda QtdProduzidaFinal,
		c2.CUSTOGERENCIAL CustoHistorico,
		PCA.CustoPorcentagem,
		isnull(PCA.CustoValor,0) CustoValor

		from ProducaoPadaria pp 
		inner join ProducaoPadariaProdutos ppp on pp.CodProduto = ppp.CodProduto 
		inner join produtos p on p.codigo = ppp.CodItem
		LEFT JOIN ProdutoCustoAdicional PCA ON PCA.CodProduto = PP.CodProduto
		left join VW_ProdutoCustoLoja c on ppp.CodItem = c.CODPRODUTO  and c.CODLOJA = @CodLoja
		LEFT join VW_ProdutoCustoLoja c2 on pp.CodProduto = c2.CODPRODUTO  and c2.CODLOJA = @CodLoja
		where pp.CodProduto in (select codigo from produtos where SUBSTRING(config,1,1) = '1')
		GROUP BY pp.CodProduto,c2.CUSTOGERENCIAL ,pp.QtdProduzida, pp.Perda, PCA.CustoValor , PCA.CustoPorcentagem 
	) as T	
	left join 
	(
		select f.CD_PRODUTO, sum(c.CUSTOGERENCIAL * f.QT_ITEM) [CustoFormulasVigencia] 
		from FN_Formulas(getdate()) f
		inner join ProducaoPadaria pp on f.CD_PRODUTO = pp.CodProduto 
		inner join VW_ProdutoCustoLoja c on f.CD_ITEM = c.CODPRODUTO  and c.CODLOJA = 1		
		GROUP BY f.CD_PRODUTO, pp.QtdProduzida, pp.Perda
	) as FV on t.CodProduto = fv.CD_PRODUTO 


) AS T2 			 
where 
(
	abs(CONVERT(FLOAT,t2.[CustoGerencialFormulasVigencia]) - CONVERT(FLOAT,[CustoGerencialProducaoPadaria])) > (CONVERT(FLOAT,[CustoGerencialProducaoPadaria]) * 0.1)
	OR 
	abs(CONVERT(FLOAT,t2.[CustoHistorico]) - CONVERT(FLOAT,[CustoGerencialProducaoPadaria])) > (CONVERT(FLOAT,[CustoGerencialProducaoPadaria]) * 0.1)
)
and t2.CodProduto not in (select distinct CD_PRODUTO from NF_ENTRADAS_PRODUTOS)
order by t2.CodProduto

*/


-- FAZ BKP DAS TABELAS QUE IREMOS ALTERAR PARA CASO DÊ ALGUM PROBLEMA POSSAMOS DESFAZER AS ALTERAÇÕES

IF (EXISTS (SELECT * 
                 FROM INFORMATION_SCHEMA.TABLES 
                 WHERE TABLE_SCHEMA = 'dbo' 
                 AND  TABLE_NAME = 'WWWBKPFormulasVigenciaErroProducaoPadaria'))
BEGIN
    drop table WWWBKPFormulasVigenciaErroProducaoPadaria
END;

IF (EXISTS (SELECT * 
                 FROM INFORMATION_SCHEMA.TABLES 
                 WHERE TABLE_SCHEMA = 'dbo' 
                 AND  TABLE_NAME = 'WWWBKPProducaoPadariaProdutosErroProducaoPadaria'))
BEGIN
    drop table WWWBKPProducaoPadariaProdutosErroProducaoPadaria
END;

select * into WWWBKPFormulasVigenciaErroProducaoPadaria from FormulasVigencia 
select * into WWWBKPProducaoPadariaProdutosErroProducaoPadaria from ProducaoPadariaProdutos 

GO 

--select * from ProdutosCorrigidosProducaoPadaria

-- DELETAR TABELA TEMPORÁRIA

IF (EXISTS (SELECT * 
                 FROM INFORMATION_SCHEMA.TABLES 
                 WHERE TABLE_SCHEMA = 'dbo' 
                 AND  TABLE_NAME = 'ProdutosCorrigidosProducaoPadaria'))
BEGIN
    drop table ProdutosCorrigidosProducaoPadaria
END;


GO

-- CRIAR TABELA TEMPORÁRIA
CREATE TABLE ProdutosCorrigidosProducaoPadaria (CodProduto FLOAT)			

GO

-- CORRIGIR BUG DE PRODUTOS QUE TEM ESTÃO COM OS ITENS DOBRADOS NA TABELA DE FÓRMULAS VIGENCIA EM RELAÇÃO A PRODUÇÃO PADARIA 

DECLARE @CodProduto AS BIGINT
DECLARE @DataFormula AS DATETIME

DECLARE rsCursor CURSOR LOCAL 
FOR 

select CodProduto,  maxDTFormula  from 
(
	SELECT CodProduto ,
	(select count(codproduto) from ProducaoPadariaProdutos where CodProduto = ppp.CodProduto ) CountProducaoPadaria,
	(select count(CD_PRODUTO) from FN_Formulas(getdate())  where CD_PRODUTO = ppp.CodProduto ) countFNFormulas,
	(select max(DtFormula) from FormulasVigencia where CodProduto = ppp.CodProduto ) maxDTFormula
	FROM ProducaoPadaria ppp
) as t
where convert(integer,CountProducaoPadaria * 2) = convert(integer,countFNFormulas)
 
OPEN rsCursor 
  
FETCH NEXT 
FROM rsCursor 
INTO @CodProduto,@DataFormula 
  
WHILE @@FETCH_STATUS = 0 
BEGIN 		  

	delete from FormulasVigencia where CodProduto = @CodProduto and DtFormula = @DataFormula

	insert into FormulasVigencia
	select CodProduto, CodItem, CONVERT(float,(ppp.QtdUtilizada / p.QtdRefUndBasica)) QtItem, 0 , 0 , @DataFormula , 1 from ProducaoPadariaProdutos ppp 
	inner join Produtos P on ppp.CodItem = p.Codigo 
	where ppp. CodProduto = @CodProduto

	INSERT INTO ProdutosCorrigidosProducaoPadaria select @CodProduto

FETCH NEXT 
FROM rsCursor 
INTO @CodProduto,@DataFormula 
END 
  
CLOSE rsCursor 
  
DEALLOCATE rsCursor

GO

-------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------

-- SCRIPT PARA CORREÇÃO DOS PRODUTOS QUE TIVERAM A FÓRMULA VIGÊNCIA REMOVIDA MAS A FÓRMULA DA PRODUÇÃO PADARIA NÃO, DAI QUANDO FOI REATIVADO FICOU COM ERRO

DECLARE @CodProduto AS BIGINT
DECLARE @DataFormula AS DATETIME

DECLARE rsCursor CURSOR LOCAL 
FOR 

select CodProduto, maxDTFormula from 
(
	SELECT CodProduto ,
	(select count(codproduto) from ProducaoPadariaProdutos where CodProduto = ppp.CodProduto ) CountProducaoPadaria,
	(select count(CD_PRODUTO) from FN_Formulas(getdate())  where CD_PRODUTO = ppp.CodProduto ) countFNFormulas,
	(select count(CodProduto) from FormulasVigencia where CodProduto = ppp.CodProduto ) countFNFormulasVigencia,
	(select max(DtFormula) from FormulasVigencia where CodProduto = ppp.CodProduto ) maxDTFormula
	FROM ProducaoPadaria ppp
) as t
where CountProducaoPadaria <> countFNFormulas and countFNFormulas = 0 and CountProducaoPadaria > 0 and countFNFormulasVigencia > 0
and CodProduto in (select codigo from produtos where substring(config,1,1) = '1')  

OPEN rsCursor 
  
FETCH NEXT 
FROM rsCursor 
INTO @CodProduto,@DataFormula 
  
WHILE @@FETCH_STATUS = 0 
BEGIN 		

	delete from FormulasVigencia where CodProduto = @CodProduto and DtFormula = @DataFormula and CodItem is null

	INSERT INTO ProdutosCorrigidosProducaoPadaria select @CodProduto

FETCH NEXT 
FROM rsCursor 
INTO @CodProduto,@DataFormula 
END 
  
CLOSE rsCursor 
  
DEALLOCATE rsCursor

GO

-------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------
-- CORRIGIR OS ERROS NA TABELA PRODUÇÃO PADARIA DEVIDO A ALTERAÇÃO DA UNIDADE DE REFERENCIA DOS PRODUTOS
-- EM ALGUNS PRODUTOS QUE JÁ FORAM SALVOS , TENTAREMOS PEGAR O VALOR QUE ESTAVA NA FÓRMULA VIGÊNCIA ANTERIOR

DECLARE @CodProduto AS FLOAT
DECLARE @CodItem AS FLOAT
DECLARE @QtdUtilizada AS decimal
DECLARE @QtItem AS float
DECLARE @QtdRefUndBasica AS MONEY
DECLARE @QtItemXQtdRefUndBasica AS float
DECLARE @qtItemMaisAntiga AS float
DECLARE @MAXDATA AS DATETIME

DECLARE rsCursor CURSOR LOCAL 
FOR 

	select 
	fv.CodProduto, 
	fv.CodItem , 
	ppp.QtdUtilizada,
	fv.QtItem , 
	p.QtdRefUndBasica , 
	fv.QtItem * p.QtdRefUndBasica QtItemXQtdRefUndBasica, 
	ISNULL((
		select top 1 fv2.QtItem 
		from FormulasVigencia fv2 
		where fv2.CodProduto = fv.CodProduto and fv2.CodItem = fv.CodItem and fv2.DtFormula < t1.maxData
		order by fv2.DtFormula desc
	),0) qtItemMaisAntiga,
	maxData

	from FormulasVigencia fv inner join
	(
		select fv.CodProduto , max(fv.DtFormula) maxData 
		from FormulasVigencia fv 
		where fv.CodProduto in (select distinct CodProduto from ProducaoPadariaProdutos )
		group by fv.CodProduto 
	) t1 on fv.CodProduto = t1.CodProduto and fv.DtFormula = t1.maxData 
	inner join ProducaoPadariaProdutos ppp on fv.CodProduto = ppp.CodProduto and fv.CodItem = ppp.CodItem 
	inner join Produtos P on fv.CodItem = p.Codigo

	where  fv.CodProduto in (select codigo from produtos where substring(config,1,1) = '1')  
	and abs(Round(ppp.QtdUtilizada,2,1) - round(fv.QtItem * p.QtdRefUndBasica,2,1)) > Round(ppp.QtdUtilizada,2,1)  * 0.1
	order by  maxData

OPEN rsCursor
  
FETCH NEXT 
FROM rsCursor 
INTO @CodProduto,@CodItem,@QtdUtilizada,@QtItem,@QtdRefUndBasica,@QtItemXQtdRefUndBasica,@qtItemMaisAntiga,@MAXDATA
  
WHILE @@FETCH_STATUS = 0 
BEGIN 		
	
	-- NESTE CASO O CLIENTE JÁ SALVOU E ESTÁ ERRADO , VAMOS TENTAR UTILIZAR O MAIS ANTIGO 
	IF  @MAXDATA > '2022-03-24 00:00:00.000' and @QtItem > ( @qtItemMaisAntiga * 900 )
		BEGIN 
		
			UPDATE FormulasVigencia SET QtItem = @qtItemMaisAntiga WHERE CodProduto = @CodProduto AND CodItem = @CodItem AND DtFormula = @MAXDATA 

			UPDATE ProducaoPadariaProdutos SET QtdUtilizada = @qtItemMaisAntiga * @QtdRefUndBasica WHERE CodProduto = @CodProduto AND CodItem = @CodItem

			INSERT INTO ProdutosCorrigidosProducaoPadaria select @CodProduto
					
		END 
	ELSE 
		BEGIN

	        UPDATE ProducaoPadariaProdutos SET QtdUtilizada = @QtItemXQtdRefUndBasica WHERE CodProduto = @CodProduto AND CodItem = @CodItem
	
		    
		END;

FETCH NEXT 
FROM rsCursor 
INTO @CodProduto,@CodItem,@QtdUtilizada,@QtItem,@QtdRefUndBasica,@QtItemXQtdRefUndBasica,@qtItemMaisAntiga,@MAXDATA
END 
  
CLOSE rsCursor 
  
DEALLOCATE rsCursor


GO

-- RECALCULAMOS OS CUSTOS NAS NOTAS PARA AJUSTAR OS CUSTOS DE ACORDO COM AS ALTERAÇÕES EM FÓRMULAS VIGÊNCIA

DECLARE @SEQUENCIA AS BIGINT
DECLARE @CD_NOTA AS FLOAT

DECLARE @TAB_PRODUTOS AS TABLE (COD_PRODUTO DECIMAL)

INSERT INTO @TAB_PRODUTOS  

select distinct CodItem from ProducaoPadariaProdutos where CodProduto in (select distinct CodProduto from ProdutosCorrigidosProducaoPadaria)

INSERT INTO @TAB_PRODUTOS

select distinct Pai from ProdutosFilhosCadDeProdutos WHERE filho in (select distinct COD_PRODUTO from @TAB_PRODUTOS)

DECLARE rsCursor CURSOR LOCAL 
FOR 

select NF.CD_NOTA from nf_entradas nf inner join NF_ENTRADAS_PRODUTOS nfp on nf.CD_NOTA = nfp.CD_NOTA 
where year(DT_ENTRADA) = 2022 
and month(DT_ENTRADA) >= 3
and 
nfp.CD_PRODUTO in (select COD_PRODUTO FROM @TAB_PRODUTOS)
group by nf.CD_NOTA 
order by nf.CD_NOTA asc

OPEN rsCursor 
  
FETCH NEXT 
FROM rsCursor 
INTO @CD_NOTA 
  
WHILE @@FETCH_STATUS = 0 
BEGIN 		
			DECLARE rsCursor2 CURSOR LOCAL 
			FOR 
			
			SELECT SEQUENCIA FROM NF_ENTRADAS_PRODUTOS WHERE CD_NOTA = @CD_NOTA

			OPEN rsCursor2 
  
			FETCH NEXT 
			FROM rsCursor2 
			INTO @SEQUENCIA 
  
			WHILE @@FETCH_STATUS = 0 
			BEGIN

				    UPDATE NF_ENTRADAS_PRODUTOS set CUSTO_GERENCIAL = CUSTO_GERENCIAL where CD_NOTA = @CD_NOTA AND SEQUENCIA = @SEQUENCIA AND CD_PRODUTO IN (select COD_PRODUTO FROM @TAB_PRODUTOS)

				FETCH NEXT 
				FROM rsCursor2
				INTO @SEQUENCIA
			END 
  
		CLOSE rsCursor2  
		DEALLOCATE rsCursor2

	FETCH NEXT 
	FROM rsCursor 
	INTO @CD_NOTA
END 
  
CLOSE rsCursor 
  
DEALLOCATE rsCursor

GO
 

--select * from ProdutosCorrigidosProducaoPadaria2

-- DELETAR TABELA TEMPORÁRIA 2

IF (EXISTS (SELECT * 
                 FROM INFORMATION_SCHEMA.TABLES 
                 WHERE TABLE_SCHEMA = 'dbo' 
                 AND  TABLE_NAME = 'ProdutosCorrigidosProducaoPadaria2'))
BEGIN
    drop table ProdutosCorrigidosProducaoPadaria2
END;


GO

-- CRIAR TABELA TEMPORÁRIA
CREATE TABLE ProdutosCorrigidosProducaoPadaria2 (CodProduto FLOAT)			

GO

-- DEVIDO AS ALTERAÇÕES PODEMOS VIR A TER ALGUM PRODUTO QUE FICOU COM DIVERGÊNCIA NO CUSTO GERENCIAL DEVIDO A ALTERAÇÃO DE ALGUM OUTRO INGREDIENTE QUE FOI RECALCULADO
DECLARE @CodLoja AS INT SET @CodLoja = 1

INSERT INTO ProdutosCorrigidosProducaoPadaria2
SELECT CodProduto FROM
(
	select CodProduto,
	(case when CustoPorcentagem > 0 then [CustoGerencialProducaoPadaria] + (([CustoGerencialProducaoPadaria] * CustoPorcentagem) / 100) else [CustoGerencialProducaoPadaria] + CustoValor end) / QtdProduzidaFinal [CustoGerencialProducaoPadaria],
	(case when CustoPorcentagem > 0 then [CustoFormulasVigencia] + (([CustoFormulasVigencia] * CustoPorcentagem) / 100) else [CustoFormulasVigencia] + CustoValor end) / QtdProduzidaFinal [CustoGerencialFormulasVigencia],
	CustoHistorico 
	from 
	(
		select 

		pp.CodProduto,
		sum(c.CUSTOGERENCIAL * (ppp.QtdUtilizada / p.QtdRefUndBasica)) [CustoGerencialProducaoPadaria],
		pp.QtdProduzida - pp.Perda QtdProduzidaFinal,
		c2.CUSTOGERENCIAL CustoHistorico,
		PCA.CustoPorcentagem,
		isnull(PCA.CustoValor,0) CustoValor

		from ProducaoPadaria pp 
		inner join ProducaoPadariaProdutos ppp on pp.CodProduto = ppp.CodProduto 
		inner join produtos p on p.codigo = ppp.CodItem
		LEFT JOIN ProdutoCustoAdicional PCA ON PCA.CodProduto = PP.CodProduto
		left join VW_ProdutoCustoLoja c on ppp.CodItem = c.CODPRODUTO  and c.CODLOJA = @CodLoja
		LEFT join VW_ProdutoCustoLoja c2 on pp.CodProduto = c2.CODPRODUTO  and c2.CODLOJA = @CodLoja
		where pp.CodProduto in (select codigo from produtos where SUBSTRING(config,1,1) = '1')
		GROUP BY pp.CodProduto,c2.CUSTOGERENCIAL ,pp.QtdProduzida, pp.Perda, PCA.CustoValor , PCA.CustoPorcentagem 
	) as T	
	left join 
	(
		select f.CD_PRODUTO, sum(c.CUSTOGERENCIAL * f.QT_ITEM) [CustoFormulasVigencia] 
		from FN_Formulas(getdate()) f
		inner join ProducaoPadaria pp on f.CD_PRODUTO = pp.CodProduto 
		inner join VW_ProdutoCustoLoja c on f.CD_ITEM = c.CODPRODUTO  and c.CODLOJA = 1		
		GROUP BY f.CD_PRODUTO, pp.QtdProduzida, pp.Perda
	) as FV on t.CodProduto = fv.CD_PRODUTO 


) AS T2 			 
where 
(
	abs(CONVERT(FLOAT,t2.[CustoGerencialFormulasVigencia]) - CONVERT(FLOAT,[CustoGerencialProducaoPadaria])) > (CONVERT(FLOAT,[CustoGerencialProducaoPadaria]) * 0.1)
	OR 
	abs(CONVERT(FLOAT,t2.[CustoHistorico]) - CONVERT(FLOAT,[CustoGerencialProducaoPadaria])) > (CONVERT(FLOAT,[CustoGerencialProducaoPadaria]) * 0.1)
)
and t2.CodProduto not in (select distinct CD_PRODUTO from NF_ENTRADAS_PRODUTOS)
order by t2.CodProduto



-- RECALCULAR CUSTO DAS NOTAS COM OS PRODUTOS ACIMA

DECLARE @SEQUENCIA AS BIGINT
DECLARE @CD_NOTA AS FLOAT

DECLARE @TAB_PRODUTOS AS TABLE (COD_PRODUTO DECIMAL)

INSERT INTO @TAB_PRODUTOS  

select distinct CodItem from ProducaoPadariaProdutos where CodProduto in (select distinct CodProduto from ProdutosCorrigidosProducaoPadaria2)

INSERT INTO @TAB_PRODUTOS

select distinct Pai from ProdutosFilhosCadDeProdutos WHERE filho in (select distinct COD_PRODUTO from @TAB_PRODUTOS)

DECLARE rsCursor CURSOR LOCAL 
FOR 

select NF.CD_NOTA from nf_entradas nf inner join NF_ENTRADAS_PRODUTOS nfp on nf.CD_NOTA = nfp.CD_NOTA 
where year(DT_ENTRADA) = 2022 
and month(DT_ENTRADA) >= 6
and 
nfp.CD_PRODUTO in (select COD_PRODUTO FROM @TAB_PRODUTOS)
group by nf.CD_NOTA 
order by nf.CD_NOTA asc

OPEN rsCursor 
  
FETCH NEXT 
FROM rsCursor 
INTO @CD_NOTA 
  
WHILE @@FETCH_STATUS = 0 
BEGIN 		
			DECLARE rsCursor2 CURSOR LOCAL 
			FOR 
			
			SELECT SEQUENCIA FROM NF_ENTRADAS_PRODUTOS WHERE CD_NOTA = @CD_NOTA

			OPEN rsCursor2 
  
			FETCH NEXT 
			FROM rsCursor2 
			INTO @SEQUENCIA 
  
			WHILE @@FETCH_STATUS = 0 
			BEGIN

				    UPDATE NF_ENTRADAS_PRODUTOS set CUSTO_GERENCIAL = CUSTO_GERENCIAL where CD_NOTA = @CD_NOTA AND SEQUENCIA = @SEQUENCIA AND CD_PRODUTO IN (select COD_PRODUTO FROM @TAB_PRODUTOS)

				FETCH NEXT 
				FROM rsCursor2
				INTO @SEQUENCIA
			END 
  
		CLOSE rsCursor2  
		DEALLOCATE rsCursor2

	FETCH NEXT 
	FROM rsCursor 
	INTO @CD_NOTA
END 
  
CLOSE rsCursor 
  
DEALLOCATE rsCursor







```
