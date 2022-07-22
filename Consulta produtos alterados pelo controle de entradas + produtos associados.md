``` SQL

DECLARE @DataVersao as datetime
set @DataVersao = '2022-07-14 11:00:00'

SELECT 
ProdutosAssociados.CD_PRODUTO [CÓDIGO] 
, (SELECT DescricaoReduzida FROM PRODUTOS WHERE CODIGO = ProdutosAssociados.CD_PRODUTO) [PRODUTO] 
, ProdutosAssociados.valorProduto [VALOR PRODUTO]
, ProdutosAlterados.valorProduto [VALOR CONTROLE DE ENTRADAS]
FROM 

(
	SELECT DISTINCT (SELECT CD_ASSOCIADO FROM PRODUTOS WHERE CODIGO = NFP.CD_PRODUTO) CD_ASSOCIADO , PL.valorProduto
	FROM 
	NF_ENTRADAS NF 
	INNER JOIN NF_ENTRADAS_PRODUTOS NFP ON NF.CD_NOTA = NFP.CD_NOTA
	INNER JOIN ProdutoLojas PL ON NFP.CD_PRODUTO = PL.codProduto AND NF.CodLoja = PL.codLoja 
	WHERE 
	NF.CD_NOTA IN (SELECT CD_NOTA FROM NF_ENTRADAS WHERE DT_ENTRADA >= @DataVersao)
	AND NFP.cd_produto in (select codigo from produtos where CD_ASSOCIADO in (select codigo from PRODUTOS_ASSOCIADOS))
	AND NFP.cd_produto in (	select CD_PRODUTO from PRECOS_ALTERADOS where DATA_ALTERACAO >= @DataVersao)
) ProdutosAlterados

INNER JOIN 

(

	select DISTINCT PA.CD_PRODUTO , PL.valorProduto , (SELECT CODIGO FROM PRODUTOS_ASSOCIADOS WHERE CODIGO IN (SELECT CD_ASSOCIADO FROM PRODUTOS WHERE CODIGO = PA.CD_PRODUTO)) CD_ASSOCIADO
	from PRECOS_ALTERADOS PA
	INNER JOIN ProdutoLojas PL ON PA.CD_PRODUTO = PL.codProduto AND PA.CODLOJA = PL.codLoja 
	where 
	DATA_ALTERACAO >= @DataVersao
	and cd_produto in (select codigo from produtos where CD_ASSOCIADO in (select codigo from PRODUTOS_ASSOCIADOS))
	and cd_produto in (SELECT CODIGO FROM PRODUTOS WHERE SUBSTRING(CONFIG,1,1) = '1')

) ProdutosAssociados

on ProdutosAlterados.CD_ASSOCIADO = ProdutosAssociados.CD_ASSOCIADO

WHERE ProdutosAlterados.valorProduto <> ProdutosAssociados.valorProduto

order by abs(ProdutosAssociados.valorProduto - ProdutosAlterados.valorProduto) DESC

```
