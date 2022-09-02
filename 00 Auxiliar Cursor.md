``` SQL


  DECLARE @Codigo TINYINT

  DECLARE rsCursor CURSOR LOCAL FOR 
  
  SELECT 1 
    
  OPEN rsCursor 
  FETCH NEXT FROM rsCursor INTO @Codigo
  WHILE @@FETCH_STATUS = 0 
  BEGIN 

	 Print convert(nvarchar(max),@Codigo)
     FETCH NEXT FROM rsCursor INTO @Codigo
  END 
 
  CLOSE rsCursor 
  DEALLOCATE rsCursor
  

``` 
