``` SQL

-- ARRUMAR AS CONFIGURAÇÕES DOS PRODUTOS
      
	   UPDATE P set  FatorCustoDePaiParaFilho = 1, VincularCustoGerencialTotalItens = 0 from produtos p where FatorCustoDePaiParaFilho = 1 and VincularCustoGerencialTotalItens = 1
       AND codigo in (select distinct CD_PRODUTO from NF_ENTRADAS_PRODUTOS )
       
GO

	   update produtos set FatorCustoDePaiParaFilho = 0, VincularCustoGerencialTotalItens = 1 where VincularCustoGerencialTotalItens = 1 and FatorCustoDePaiParaFilho = 1	   
	   
GO

-- DELETAR OS REGISTROS DE CADASTRO DE PRODUTOS NA TABELA CUSTO HISTÓRICO

	ALTER TABLE CustoHistorico DISABLE TRIGGER ALL

	DELETE from CustoHistorico where year(DtInsercao) = 2022 AND ORIGEM = 5
	and month(DtInsercao) >= 3 and AlteraCusto = 1 

GO

	DELETE from CustoHistorico where year(DtInsercao) = 2022 AND ORIGEM = 3
	and month(DtInsercao) >= 3 and AlteraCusto = 1 

GO

	DELETE from CustoHistorico where CodNota IN (
		select DISTINCT NF.CD_NOTA from nf_entradas nf inner join NF_ENTRADAS_PRODUTOS nfp on nf.CD_NOTA = nfp.CD_NOTA 
		where year(DT_ENTRADA) = 2022 and month(DT_ENTRADA) >= 3 
		and nfp.CD_PRODUTO in (select CD_ITEM from formulas where CD_PRODUTO in (select codigo from produtos where VincularCustoGerencialTotalItens = 1))
		and nfp.CD_PRODUTO in (select CD_ITEM from formulas where CD_PRODUTO in (select cd_produto from formulas group by cd_produto having count(cd_item) > 1))
	)


	ALTER TABLE CustoHistorico ENABLE TRIGGER ALL


GO

-- RODAR O SCRIPT PARA CORREÇÃO DAS NOTAS

		DECLARE @SEQUENCIA AS BIGINT
		DECLARE @CD_NOTA AS FLOAT

		DECLARE rsCursor CURSOR LOCAL 
		FOR 
			select DISTINCT NF.CD_NOTA from nf_entradas nf inner join NF_ENTRADAS_PRODUTOS nfp on nf.CD_NOTA = nfp.CD_NOTA 
			where year(DT_ENTRADA) = 2022 and month(DT_ENTRADA) >= 3 
			and nfp.CD_PRODUTO in (select CD_ITEM from formulas where CD_PRODUTO in (select codigo from produtos where VincularCustoGerencialTotalItens = 1))
			and nfp.CD_PRODUTO in (select CD_ITEM from formulas where CD_PRODUTO in (select cd_produto from formulas group by cd_produto having count(cd_item) > 1))

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


					 update NF_ENTRADAS_PRODUTOS set CUSTO_GERENCIAL = CUSTO_GERENCIAL where CD_NOTA = @CD_NOTA AND SEQUENCIA = @SEQUENCIA


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

-- EXECUTAR SCRIPT PARA CORREÇÃO DO CADASTRO DE PRODUTOS


DECLARE @CodProduto AS BIGINT
DECLARE @DataFormula AS DATETIME

DECLARE rsCursor CURSOR LOCAL 
FOR 

	SELECT * FROM 
	(
		select CodProduto , DtFormula from FormulasVigencia 
		where 
		year(DtFormula) = 2022 and month(DtFormula) >= 3 
		AND CodProduto IN (SELECT CODIGO FROM PRODUTOS WHERE FatorCustoDePaiParaFilho = 1)
		group by CodProduto, DtFormula 


		UNION ALL

		select CodItem , DtFormula from FormulasVigencia 
		where 
		year(DtFormula) = 2022 and month(DtFormula) >= 3 
		AND CodProduto IN (SELECT CODIGO FROM PRODUTOS WHERE VincularCustoGerencialTotalItens = 1)
		group by CodItem, DtFormula 
	) AS TABELA
	order by DtFormula ASC

OPEN rsCursor 
  
FETCH NEXT 
FROM rsCursor 
INTO @CodProduto,@DataFormula 
  
WHILE @@FETCH_STATUS = 0 
BEGIN 		

    
	           DECLARE @CodLoja as integer
		       DECLARE rsCursor2 CURSOR LOCAL 
				FOR 
			
				SELECT CODIGO FROM LOJAS

				OPEN rsCursor2 
  
				FETCH NEXT 
				FROM rsCursor2 
				INTO @CodLoja 
  
				WHILE @@FETCH_STATUS = 0 
				BEGIN


				   EXEC [SP_InserirCustoHistorico] NULL,@CodProduto,NULL,NULL,@DataFormula,@CodLoja,0,0,0,3,1,1,0,0,0,0


					FETCH NEXT 
					FROM rsCursor2
					INTO @CodLoja
				END 
  
			CLOSE rsCursor2  
			DEALLOCATE rsCursor2



    FETCH NEXT 
    FROM rsCursor 
	INTO @CodProduto,@DataFormula 
END 
  
CLOSE rsCursor 
  
DEALLOCATE rsCursor

GO

-- EXECUTAR SCRIPT PARA CORREÇÃO DOS PRODUTOS ZERADOS
--------------------------------------------------------------------------------------------------------------

ALTER TABLE CustoHistorico DISABLE TRIGGER ALL

DELETE from CustoHistorico where year(DtCusto) = 2022 and month(DtCusto) >= 3
and Origem = 3 AND CustoGerencial = 0

ALTER TABLE CustoHistorico ENABLE TRIGGER ALL

GO

-- RODAR O SCRIPT PARA CORREÇÃO DO CUSTO MÉDIO

        DECLARE @CodOperador AS INT 
         
        SET @CodOperador = isnull((SELECT top 1 codigo FROM operador WHERE nome like '%telecon%') ,1) 
         
        ALTER TABLE CustoHistorico DISABLE TRIGGER ALL 
         
        INSERT INTO CustoHistorico  
         
        SELECT  
        CustoHistorico.CodLoja  
        , CustoHistorico.CodProduto 
        , null 
        , 5 
        , GETDATE() 
        , CustoHistorico.CustoGerencial 
        , CustoHistorico.CustoGerencial
        , CustoHistorico.CustoMargem 
        , GETDATE() 
        , @CodOperador  
        , null 
        , null 
        , 1 
        , CustoHistorico.ValorBaseStRetido 
        , CustoHistorico.AliquotaIcmsStRetido 
        , CustoHistorico.AliquotaFcpStRetido 
        , CustoHistorico.ReducaoBaseStRetido 
         
        FROM  VW_ProdutoCustoLoja CustoHistorico 
        Where CustoHistorico.CustoMedio <> CustoHistorico.CustoGerencial 
        AND CustoHistorico.CustoGerencial <> 0 
        AND ABS(CustoHistorico.CustoMedio - CustoHistorico.CustoGerencial) > (CustoHistorico.CustoGerencial * 0.5) 
        AND ( CustoHistorico.CODPRODUTO IN ( SELECT DISTINCT CD_ITEM FROM FN_Formulas(GETDATE())) or CustoHistorico.CODPRODUTO IN ( SELECT DISTINCT CD_PRODUTO FROM FN_Formulas(GETDATE()))) 
         
       ALTER TABLE CustoHistorico ENABLE TRIGGER ALL 




  
```  
