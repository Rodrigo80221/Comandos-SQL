
``` SQL
SET NOCOUNT ON 

IF (select COUNT(*) from Formas_Pgto where Config = '310') = 1
BEGIN

	PRINT 'CRIOU A NOVA FORMA DE PAGAMENTO'

	-- CADASTRA A NOVA FORMA DE PAGAMENTO
	insert into Formas_Pgto
	select 
	(select max(codigo) + 1 from Formas_Pgto)
	,'CARTAO REDE SUPER'
	,Config
	,DESCONTO
	,TROCO_MAXIMO
	,3 -- (select max(ordem) + 1 from Formas_Pgto)
	,CodFormaPagamentoFinanceiro
	,CodFormaPgtoPdvTerceiros
	,ExibirMenuAdmCartoes
	,ParticipaFormulaValAlertaSangria
	,ContraValeControladoBarras
	,AbreGaveta
	,PermiteAlteracao
	,TipoDebito
	,SincronizaComPDV
	,CodIndPresenca
	,CodCliIntermed
	,CodCliTransportador
	from Formas_Pgto where Config = '310'

	-- CADASTRA CONTAS DO FINANCEIRO 
	INSERT INTO FormasPgtoLojasFinanceiro
	select 
	 (select codigo from Formas_Pgto where NOME = 'CARTAO REDE SUPER')
	,CodLoja
	,CodContaVenda
	,CodContaMovVenda
	,CodContaRecargaCelular
	,CodContaMovRecarga
	,CodContaContraValeGerado
	,CodContaMovContraValeGerado
	,CodContaQuebraCaixa
	,CodContaMovQuebraCaixa
	,CodContaPagamentoConvenio
	,CodContaMovPagamentoConvenio
	,CodContaFidelidade
	,CodContaMovFidelidade
	from FormasPgtoLojasFinanceiro where CodFormaPgto in (select codigo from Formas_Pgto where Config = '310')

	PRINT 'ATUALIZOU A NOVA FORMA DE PAGAMENTO EM TODOS OS CAIXAS'

	-- MARCAR EM TODOS OS CAIXAS 
	DECLARE @COD_FORMAPGTO AS INTEGER = (select codigo from Formas_Pgto where NOME = 'CARTAO REDE SUPER')
	DECLARE @COD_PDV AS INTEGER
	DECLARE NOVOS CURSOR LOCAL   
	FOR  select distinct caixa from Formas_Pgto_ECF   
	OPEN NOVOS;   
	FETCH NEXT FROM NOVOS INTO @COD_PDV;   
	WHILE @@FETCH_STATUS = 0   
		BEGIN   
			INSERT INTO Formas_Pgto_ECF 
			SELECT 
			(select MAX(CODIGO) + 1 from Formas_Pgto_ECF) , @COD_PDV, @COD_FORMAPGTO, @COD_FORMAPGTO

			FETCH NEXT FROM NOVOS INTO @COD_PDV;   
		END   
	CLOSE NOVOS;   
	DEALLOCATE NOVOS;  

	PRINT 'ALTEROU OS PACKS VIRTUAIS'

	-- ALTERA O MODELO DO PACK VIRTUAL PARA MELHORAR A VISIBILIDADE NO PDV 
	ALTER TABLE PackVirtual DISABLE TRIGGER ALL
	UPDATE PackVirtual SET ModeloPack = 3 WHERE DESCRICAO = 'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 0.88)'
	UPDATE PackVirtual SET ModeloPack = 3 WHERE DESCRICAO = 'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 0.99)'
	UPDATE PackVirtual SET ModeloPack = 3 WHERE DESCRICAO = 'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 3.29)'
	UPDATE PackVirtual SET ModeloPack = 3 WHERE DESCRICAO = 'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 3.99)'
	UPDATE PackVirtual SET ModeloPack = 3 WHERE DESCRICAO = 'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 5.99)'
	UPDATE PackVirtual SET ModeloPack = 3 WHERE DESCRICAO = 'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 18.90)'
	ALTER TABLE PackVirtual ENABLE TRIGGER ALL

	PRINT 'REMOVEU OS BINS '

	-- DELETA AS BINS 
	ALTER TABLE PackVirtualFormasPgtoTEF DISABLE TRIGGER ALL
	delete from PackVirtualFormasPgtoTEF 
	WHERE CODPACK IN (SELECT CODIGO FROM PackVirtual WHERE Descricao IN (
	'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 0.88)'
	,'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 0.99)'
	,'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 3.29)'
	,'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 3.99)'
	,'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 5.99)'
	,'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 18.90)'
	))
	ALTER TABLE PackVirtualFormasPgtoTEF ENABLE TRIGGER ALL

	ALTER TABLE PackVirtualFormasPgtoTEFBins DISABLE TRIGGER ALL
	delete from PackVirtualFormasPgtoTEFBins  
	WHERE CODPACK IN (SELECT CODIGO FROM PackVirtual WHERE Descricao IN (
	'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 0.88)'
	,'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 0.99)'
	,'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 3.29)'
	,'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 3.99)'
	,'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 5.99)'
	,'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 18.90)'
	))
	ALTER TABLE PackVirtualFormasPgtoTEFBins ENABLE TRIGGER ALL

	PRINT 'ALTEROU AS FORMAS DE PAGAMENTOS NOS PACKS '

	-- DELETA AS FORMAS DE PAGAMENTO
	ALTER TABLE PackVirtualFormasPgto DISABLE TRIGGER ALL
	delete from PackVirtualFormasPgto WHERE CODPACK IN (SELECT CODIGO FROM PackVirtual WHERE Descricao = 'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 0.88)')
	delete from PackVirtualFormasPgto WHERE CODPACK IN (SELECT CODIGO FROM PackVirtual WHERE Descricao = 'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 0.99)')
	delete from PackVirtualFormasPgto WHERE CODPACK IN (SELECT CODIGO FROM PackVirtual WHERE Descricao = 'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 3.29)')
	delete from PackVirtualFormasPgto WHERE CODPACK IN (SELECT CODIGO FROM PackVirtual WHERE Descricao = 'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 3.99)')
	delete from PackVirtualFormasPgto WHERE CODPACK IN (SELECT CODIGO FROM PackVirtual WHERE Descricao = 'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 5.99)')
	delete from PackVirtualFormasPgto WHERE CODPACK IN (SELECT CODIGO FROM PackVirtual WHERE Descricao = 'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 18.90)')
	ALTER TABLE PackVirtualFormasPgto ENABLE TRIGGER ALL

	ALTER TABLE PackVirtualFormasPgto DISABLE TRIGGER ALL
	DECLARE @COD_FORMAPGTO2 AS INTEGER = (select codigo from Formas_Pgto where NOME = 'CARTAO REDE SUPER')
	INSERT INTO PackVirtualFormasPgto SELECT CODIGO, @COD_FORMAPGTO2 FROM PackVirtual WHERE Descricao = 'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 0.88)'
	INSERT INTO PackVirtualFormasPgto SELECT CODIGO, @COD_FORMAPGTO2 FROM PackVirtual WHERE Descricao = 'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 0.99)'
	INSERT INTO PackVirtualFormasPgto SELECT CODIGO, @COD_FORMAPGTO2 FROM PackVirtual WHERE Descricao = 'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 3.29)'
	INSERT INTO PackVirtualFormasPgto SELECT CODIGO, @COD_FORMAPGTO2 FROM PackVirtual WHERE Descricao = 'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 3.99)'
	INSERT INTO PackVirtualFormasPgto SELECT CODIGO, @COD_FORMAPGTO2 FROM PackVirtual WHERE Descricao = 'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 5.99)'
	INSERT INTO PackVirtualFormasPgto SELECT CODIGO, @COD_FORMAPGTO2 FROM PackVirtual WHERE Descricao = 'CARTÃO REDE MAIO/JUNHO DE 2023 (R$ 18.90)'
	ALTER TABLE PackVirtualFormasPgto ENABLE TRIGGER ALL

	PRINT 'ALTERAÇÃO FINALIZADA ENVIE UMA EXPORTAÇÃO PARA OS PDVS '

END;

```

