``` sql

CREATE TABLE BARRAROBBOTEL (BARRAS NVARCHAR(20))

SELECT * FROM BARRAROBBOTEL

INSERT INTO BARRAROBBOTEL (BARRAS) VALUES ('7891097001062')


select 
'UPDATE Produtos SET ' ,
'ICMS = ' + '''' + CONVERT(NVARCHAR(MAX),A.Descricao) + '''' + ',', -- nvarchar
'MVA = ' + CONVERT(NVARCHAR(MAX),P.MVA) + ',', -- money
'ICMS_COMPRA = ' + CONVERT(NVARCHAR(MAX),isnull(P.ICMS_COMPRA,0.00)) + ',', -- money
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
+ 'where Produtos.codigo = ''' + CONVERT(NVARCHAR(MAX),Pb.BARRAS) + ''' '
from produtos p
inner join PRODUTO_BARRAS pb on pb.CD_PRODUTO = P.Codigo
inner join AliquotasPisCofins apc on apc.Codigo = p.CodAliquotaPisCofins  
inner join Aliquotas a on a.Codigo = p.Aliquota 
inner join BARRAROBBOTEL robbo on pb.Barras = robbo.barras



/*
TABELA DE PRODUTOS DO ROBBOTEL

CREATE TABLE Produtos (
   Codigo nVarChar(20) Not Null,
   Nome nVarChar(50) Not Null,
   Valor Money Not Null,
   [CD_Informacao] Int Null,
   [Texto_Adicional] nVarChar(1024) Null,
   [Arquivo_Audio] nVarChar(256) Null,
   Promocao Bit Null,
   Icms nVarChar(20) Null,
   MVA Decimal(18, 2) Null,
   AliquotaIcmsNFSaidas Decimal(18, 2) Null,
   [ICMS_COMPRA] Decimal(18, 2) Null,
   CFOP nVarChar(4) Null,
   AliquotaICMSST Decimal(18, 2) Null,
   AliquotaFCPST Decimal(18, 2) Null,
   [BASE_REDUZIDA_ICMS] Decimal(18, 2) Null,
   BaseReduzidaST Decimal(18, 2) Null,
   ST nVarChar(3) Null,
   IPI Decimal(18, 3) Null,
   PisCofins nVarChar(20) Null,
   NaturezaReceita nVarChar(20) Null,
   NCM nVarChar(20) Null,
   [EX_Tipi] nVarChar(3) Null,
   CodigoCest nVarChar(20) Null,
   FundoCombateaPobreza Bit Null, 
   CONSTRAINT PK_Produtos Primary Key (
      Codigo
      )
   )

GO 
CREATE NONCLUSTERED INDEX idx_produtos On Produtos
      (
      Nome
      )


*/

```
