1、Install the SqlConnect plugin

https://www.unrealengine.com/marketplace/zh-CN/product/d1e931c9fb2a459ea2f33a12823e830a

2、Use plugins


![未标题-111](https://user-images.githubusercontent.com/41780542/168425467-505c733f-cd98-454e-a0f5-64ae4e8b13ea.png)
![未标题-1](https://user-images.githubusercontent.com/41780542/168425469-c2960909-2fe9-4e8b-9d7b-a64e445e63fb.png)


Features:

Support local SQL server and cloud database connection (eg: Azure Database for SQL)


Support C++ or pure blueprint calls


Synchronous and asynchronous support


used in C++


	sql = USqlConnectSubsystem::Get();
 
	Results = NewObject<USqlResult>();

	/////////////////////////// synchronous connection example ///////////////////////////////

	sql->Connect("127.0.0.1", 3306, "username", "pw", "TableName", Results);
	if (!Results->IsSucceed)
	{
		UE_LOG(LogTemp, Warning, TEXT("SQL: ConnectError: %s"), *Results->Msg);
		return;
	}

	//Get the total number of rows
	sql->Query("SELECT count(1) FROM `testtable`", Results);
	UE_LOG(LogTemp, Warning, TEXT("SQL: Count: IsSucceed = %d, msg = %s, count = %d"), Results->IsSucceed, *Results->Msg, Results->getCount());

	//get row
	sql->Query("SELECT * FROM `testtable`", Results);
	UE_LOG(LogTemp, Warning, TEXT("SQL: rows: IsSucceed = %d, msg = %s, Num = %d"), Results->IsSucceed, *Results->Msg, Results->getRows().Num());


/////////////////////////// Asynchronous connection example ///////////////////////////////

	sql = USqlConnectSubsystem::Get();
 
	Results = NewObject<USqlResult>();
	
	const FLatentActionInfo LatentInfo(0, FMath::Rand(), TEXT("ConnectCallback"), this);
	Sql->AConnect("127.0.0.1", 3306, "username", "pw", "TableName", Results, this, LatentInfo);
 
	//Asynchronous callback function
    void APluginProjectGameModeBase::ConnectCallback()
    {
        if (!Results->IsSucceed)
        {
            UE_LOG(LogTemp, Warning, TEXT("SQL: ConnectError: %s"), *Results->Msg);
            return;
        }
    
        //Get the total number of rows
        const FLatentActionInfo LatentInfo(0, FMath::Rand(), TEXT("fetch_count_Callback"), this);
        Sql->AQuery("SELECT count(1) FROM `testtable`", Results, this, LatentInfo);
    }
    
    //Asynchronous callback function
    void APluginProjectGameModeBase::fetch_count_Callback()
    {
        UE_LOG(LogTemp, Warning, TEXT("SQL: Count: IsSucceed = %d, msg = %s, count = %d"), Results->IsSucceed, *Results->Msg, Results->getCount());
    
        //get row
        const FLatentActionInfo LatentInfo(0, FMath::Rand(), TEXT("fetch_rows_Callback"), this);
        Sql->AQuery("SELECT * FROM `testtable`", Results, this, LatentInfo);
    }
    
    //Asynchronous callback function
    void APluginProjectGameModeBase::fetch_rows_Callback()
    {
        UE_LOG(LogTemp, Warning, TEXT("SQL: rows: IsSucceed = %d, msg = %s, Num = %d"), Results->IsSucceed, *Results->Msg, Results->getRows().Num());
    }
