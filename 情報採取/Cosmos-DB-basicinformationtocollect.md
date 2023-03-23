---
title: Cosmos DB 基本情報採取
date: 2023-03-20 15:00:00
categories: [Cosmos DB,情報採取]
tags:
  - NoSQL
  - MongoDB
  - Cassandra
  - Gremlin
  - Table
---

こんにちは。Cosmos DB サポートチームです。

本記事では Cosmos DB のトラブルシュートの際に必要となる情報を案内します。

問題やご相談いただく内容により必要となる情報は異なってまいりますが、今回は共通して必要となる情報について案内します。

<!-- more -->
#　基本情報

1.Subscription Id:  サブスクリプション ID

2.Cosmos DB Account 名

3.Database 名

4.Container 名

5.Cosmos DB API (NoSQL/MongoDB/Table/Gremlin/Cassandra)

6.本番環境 (Yes/No)

### <span style="text-decoration:underline">参考</span>

![](./Cosmos-DB-basicinformationtocollect/1-577ffa1a683a4217adb87b2d7573b11e.png)
![](./Cosmos-DB-basicinformationtocollect/1-967aa06703a94f55b20ff0988eb46368.png)

#　エラーの情報

1. エラーが発生した場合の詳細な時間 (日本時間の場合その旨明記。もしくは UTC 時間）    
2. Activity ID ならびにスタックトレース含むエラーメッセージ。なお、本情報は画像のみでなくテキスト形式もあわせて送付ください。 
   Activity ID が不明な場合、Activity ID の項目を参照ください。

3. 事象の発生頻度。また以前動作していたか。
4. (お気づきの場合) エラーの発生契機

5. エラーが発生するまでの手順 
毎回事象が発生する場合、エラーが発生するまでの手順を PSR で採取ください。
https://support.microsoft.com/en-us/help/22878/windows-10-record-steps

6.  (可能であれば) アプリケーションからでなくポータルからの操作で同様にエラーが発生するかお知らせください。

#　Client ・Application 情報

1. Cosmos DB に接続しているサービス (例:  Azure Functions / Azure Web App Service / <span lang="en-US" style="background-color:white">Azure Kubernetes Service (AKS)</span> , 他)

>[!NOTE]
>問題の内容に応じて利用サービスの Subscription Id や名前などの情報を依頼することがございます。
>また問題によっては該当サービス観点でのお問い合わせを相談させていただくことがございます。

2. Cosmos DB SDK とそのバージョン情報 (.Net/.Net Core/Java/Node.js/Python 他)

3. 事象が発生する最小単位のコードおよびクエリ(内容によってはサンプルデータ)

4. (アプリケーションで Diagnostics を取得している場合)  Diagnostics  
    \*SDK により取得方法異なります。

##  Activity ID

Cosmos DB に接続・操作するアプリケーションのログで Activity ID を取得していない場合、対象の Cosmos DB アカウントに対して診断設定をすることで Log Analytics から Activity ID 含む診断ログの取得が可能です。

診断ログの設定方法は https://learn.microsoft.com/ja-jp/azure/cosmos-db/monitor-resource-logs?tabs=azure-portal を参照ください。

下記はリクエストの種類 (Query / Data Plane / Control Plane) ならびに診断ログの設定 (Azure diagnostics or Resource specific) に応じたログ取得のクエリならびにその出力結果（サンプル）となります。

### <span style="text-decoration:underline">Query Requests</span>

**Azure diagnostics**

![](./Cosmos-DB-basicinformationtocollect/1-986e736d62244c54aadc2575c381a397.png)

```
AzureDiagnostics
| where ResourceProvider =="MICROSOFT.DOCUMENTDB" and Category == "QueryRuntimeStatistics"
| project TimeGenerated, activityId_g, querytext_s,databasename_s,collectionname_s,ResourceId,ResourceType,regionname_s,authtype_s,numberofrowsreturned_s,queryexecutionstatus_s,userAgent_s,partialipaddress_s,partitionKeyRangeId_s
```

![](./Cosmos-DB-basicinformationtocollect/1-03aadcebb879419ca9f78e554f3c9ea7.png)

 
 **Resource specific**
 ![](./Cosmos-DB-basicinformationtocollect/1-70e2c3f3c1e74c3eae1b1bdea4f52028.png)

```
CDBQueryRuntimeStatistics 
|project TimeGenerated,ActivityId,AccountName,DatabaseName,CollectionName,PartitionKeyRangeId,QueryText,SourceSystem,Type,_ResourceId
```

![](./Cosmos-DB-basicinformationtocollect/1-a962ac15631049a8af1f3b0b26a60b0c.png)<br>


### <span style="text-decoration:underline">Data plane requests</span>

**Azure diagnostics**

```
AzureDiagnostics
| where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests"
//| where statusCode_s !in(200,201)
| project TimeGenerated,activityId_g,statusCode_s,ResourceId,requestCharge_s,ResourceType,OperationName,requestResourceType_s,connectionMode_s,userAgent_s,clientIpAddress_s,duration_s,requestLength_s,responseLength_s,regionname_s,databasename_s
```

![](./Cosmos-DB-basicinformationtocollect/1-f347aa6a2a6a4752b3234007b5c84aa0.png)

**Resource specific**

```
CDBDataPlaneRequests
| project TimeGenerated,ActivityId,StatusCode,_ResourceId,RequestCharge,OperationName,RequestResourceType,RequestResourceId,ConnectionMode,UserAgent,ClientIpAddress,DurationMs,RequestLength,ResponseLength,RegionName,DatabaseName,CollectionName,PartitionId,KeyType,AuthTokenType
```

![](./Cosmos-DB-basicinformationtocollect/1-c13591f5f82141d0bf75f8fc00c77a5d.png)<br>


### <span style="text-decoration:underline">Control plane requests</span>

**Azure diagnostics**

```
AzureDiagnostics
| where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="ControlPlaneRequests"
| project TimeGenerated,activityId_g,statusCode_s,ResourceId,requestCharge_s,ResourceType,OperationName,requestResourceType_s,connectionMode_s,userAgent_s,clientIpAddress_s,duration_s,requestLength_s,responseLength_s,regionname_s,databasename_s
```

 ![](./Cosmos-DB-basicinformationtocollect/1-192dfbd57fb84b41be1c158e8bad3232.png)

**Resource specific**

```
CDBControlPlaneRequests
|project TimeGenerated,ActivityId,AccountName,OperationName,OperationType,HttpStatusCode,Result,HttpMethod,ApiKind,ApiKindResourceType,ResourceUri,ResourceDetails,Type,_ResourceId
```

![](./Cosmos-DB-basicinformationtocollect/1-fe1b060f52584438a7425a749ed444b9.png)


<span style="text-decoration:underline">補足</span>

上記採取情報をテキスト形式でもまとめています。(診断ログの項目除く)

必要に応じてこちらもご活用ください。

[cosmosdbbasicinformationtocollect.txt](./Cosmos-DB-basicinformationtocollect/cosmosdbbasicinformationtocollect-1-4475d98c8b694cb4b66877c4443918cb.txt)

