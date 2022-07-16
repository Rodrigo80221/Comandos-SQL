``` sql

  
DECLARE @DataCusto DATETIME
DECLARE @CodLoja INT
DECLARE @CodProduto FLOAT

SET @DataCusto = '2022-07-07 14:27:00.000'   
SET @CodLoja = 1  
SET @CodProduto = 4762 
  
SELECT T.CodProduto   
   , SUM(T.QtItem * CustoGerencial) AS CustoGerencial   
   , SUM(T.QtItem * CustoMedio) AS CustoMedio   
   , SUM(T.QtItem * CustoMargem) AS CustoMargem      
   , ISNULL(CustoPorcentagem, 0) AS CustoPorcentagem   
   , ISNULL(CustoValor, 0) AS CustoValor   
   , (ISNULL(QtdProduzida, 0) - ISNULL(Perda, 0)) AS QtdProduzida   
  FROM (   
   SELECT F.CodProduto   
       , F.CodItem   
       , F.QtItem   
       ,    ISNULL ( 
                        ( 
                            SELECT TOP 1 CUSTOGERENCIAL
                            FROM CustoHistorico CH WITH(NOLOCK) 
                            WHERE CH.CodLoja = @CodLoja AND CH.CodProduto = F.CodItem AND CH.DtCusto <= @DataCusto And CH.AlteraCusto = 1
                            ORDER BY DtCusto DESC, DtInsercao DESC  
                        ),  
                        ( 
                            SELECT AVG(VW.CUSTOGERENCIAL) 
                            FROM VW_ProdutoCustoLoja VW 
                            WHERE VW.codproduto = F.CodItem 
                            AND VW.CodLoja = @CodLoja 
                        ) 
            ) AS CustoGerencial
       ,    ISNULL ( 
                        ( 
                            SELECT TOP 1 CustoMedio
                            FROM CustoHistorico CH WITH(NOLOCK) 
                            WHERE CH.CodLoja = @CodLoja AND CH.CodProduto = F.CodItem AND CH.DtCusto <= @DataCusto And CH.AlteraCusto = 1
                            ORDER BY DtCusto DESC, DtInsercao DESC  
                        ),  
                        ( 
                            SELECT AVG(VW.CustoMedio) 
                            FROM VW_ProdutoCustoLoja VW 
                            WHERE VW.codproduto = @CodProduto 
                            AND VW.CodLoja = @CodLoja 
                        ) 
            ) AS CustoMedio
      ,     ISNULL ( 
                        ( 
                            SELECT TOP 1 CUSTOMARGEM 
                            FROM CustoHistorico CH WITH(NOLOCK) 
                            WHERE CH.CodLoja = @CodLoja AND CH.CodProduto = F.CodItem AND CH.DtCusto <= @DataCusto And CH.AlteraCusto = 1
                            ORDER BY DtCusto DESC, DtInsercao DESC  
                        ),  
                        ( 
                            SELECT AVG(VW.CUSTOMARGEM) 
                            FROM VW_ProdutoCustoLoja VW 
                            WHERE VW.codproduto = F.CodItem 
                            AND VW.CodLoja = @CodLoja 
                        ) 
            ) AS CustoMargem
   FROM FormulasVigencia F   
   INNER JOIN Produtos P ON (P.codigo = F.CodItem)   
   WHERE F.CodProduto IN (   
           SELECT DISTINCT (CD_PRODUTO)   
           FROM dbo.FN_Formulas(@DataCusto) FF
           INNER JOIN Produtos P ON (FF.CD_PRODUTO = P.Codigo)   
           WHERE FF.CD_ITEM = @CodProduto   
               AND P.VincularCustoGerencialTotalItens = 1   
               AND CONVERT(DATETIME, F.DtFormula) = ISNULL((   
                       SELECT TOP 1 CONVERT(DATETIME, F2.DtFormula)   
                       FROM FormulasVigencia F2   
                       WHERE F2.CodProduto = FF.CD_PRODUTO   
                           AND F2.DtFormula <= @DataCusto   
                       ORDER BY F2.DtFormula DESC   
                       ), CONVERT(DATETIME, GETDATE()))   
          ) 
   ) AS T   
  LEFT JOIN ProdutoCustoAdicional PCA ON PCA.CodProduto = T.CodProduto   
  LEFT JOIN ProducaoPadaria PP ON PP.CodProduto = T.CodProduto   
  GROUP BY T.CodProduto   
   , CustoPorcentagem   
   , CustoValor   
   , QtdProduzida   
   , Perda


```
