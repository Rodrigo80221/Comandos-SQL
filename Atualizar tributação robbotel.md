``` sql

CREATE TABLE BARRAROBBOTEL (BARRAS NVARCHAR(20))

SELECT * FROM BARRAROBBOTEL

INSERT INTO BARRAROBBOTEL (BARRAS) VALUES ('7891097001062')




select 
'UPDATE Produtos SET ' ,
'ICMS = ' + '''' + CONVERT(NVARCHAR(MAX),A.Descricao) + '''' + ',', -- nvarchar
'MVA = ' + CONVERT(NVARCHAR(MAX),P.MVA) + ',', -- money
'ICMS_COMPRA = ' + CONVERT(NVARCHAR(MAX),P.ICMS_COMPRA) + ',', -- money
'ALIQUOTAICMSNFSAIDAS = ' + CONVERT(NVARCHAR(MAX),P.ALIQUOTAICMSNFSAIDAS) + ',', -- money
'BASE_REDUZIDA_ICMS = ' + CONVERT(NVARCHAR(MAX),P.BASE_REDUZIDA_ICMS) + ',', -- money
'ST = ' + '''' + CONVERT(NVARCHAR(MAX),P.ST) + '''' + ',', -- char
'CFOP = ' + CONVERT(NVARCHAR(MAX),P.CFOP) + ',', -- int
'AliquotaICMSST = ' + CONVERT(NVARCHAR(MAX),P.AliquotaICMSST) + ',', -- AliquotaICMSST
'FundoCombateaPobreza = ' + CONVERT(NVARCHAR(MAX),P.FundoCombateaPobreza) + ',', -- bit
'BaseReduzidaST = ' + CONVERT(NVARCHAR(MAX),P.BaseReduzidaST) + ',', -- money
'IPI = ' + CONVERT(NVARCHAR(MAX),P.IPI) + ',', -- money
'PisCofins = '+ '''' + CONVERT(NVARCHAR(MAX),apc.CofinsEntrada)+ '''' + ',', -- nvarchar
'NATUREZARECEITA = ' + CONVERT(NVARCHAR(MAX),P.NATUREZARECEITA) + ',', -- int
'NCM = '+ '''' + CONVERT(NVARCHAR(MAX),P.NCM)+ '''' + ',', -- nvarchar
'EX_TIPI = '+ '''' + CONVERT(NVARCHAR(MAX),P.EX_TIPI) + '''' + ',', -- nvarchar
'CodigoCest = '+ '''' + CONVERT(NVARCHAR(MAX),P.CodigoCest) + '''', -- nvarchar
+ 'where Produtos.codigo in (select top 1 Cd_produto from PRODUTO_BARRAS where Barras = ''' + CONVERT(NVARCHAR(MAX),Pb.BARRAS) + ''') '
from produtos p
inner join PRODUTO_BARRAS pb on pb.CD_PRODUTO = P.Codigo
inner join AliquotasPisCofins apc on apc.Codigo = p.CodAliquotaPisCofins  
inner join Aliquotas a on a.Codigo = p.Aliquota 
inner join BARRAROBBOTEL robbo on pb.Barras = robbo.barras


```
