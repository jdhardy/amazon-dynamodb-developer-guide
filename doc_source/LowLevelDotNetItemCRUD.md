# Working with Items: \.NET<a name="LowLevelDotNetItemCRUD"></a>


+ [Putting an Item](#PutItemLowLevelAPIDotNet)
+ [Getting an Item](#GetItemLowLevelDotNET)
+ [Updating an Item](#UpdateItemLowLevelDotNet)
+ [Atomic Counter](#AtomicCounterLowLevelDotNet)
+ [Deleting an Item](#DeleteMidLevelDotNet)
+ [Batch Write: Putting and Deleting Multiple Items](#BatchWriteLowLevelDotNet)
+ [Batch Get: Getting Multiple Items](#BatchGetLowLevelDotNet)
+ [Example: CRUD Operations Using the AWS SDK for \.NET Low\-Level API](LowLevelDotNetItemsExample.md)
+ [Example: Batch Operations Using AWS SDK for \.NET Low\-Level API](batch-operation-lowlevel-dotnet.md)
+ [Example: Handling Binary Type Attributes Using the AWS SDK for \.NET Low\-Level API](LowLevelDotNetBinaryTypeExample.md)

You can use the AWS SDK for \.NET low\-level API to perform typical create, read, update, and delete \(CRUD\) operations on an item in a table\.

The following are the common steps you follow to perform data CRUD operations using the \.NET low\-level API\. 

1. Create an instance of the `AmazonDynamoDBClient` class \(the client\)\.

1. Provide the operation specific required parameters in a corresponding request object\.

   For example, use the `PutItemRequest` request object when uploading an item and use the `GetItemRequest` request object when retrieving an existing item\. 

   You can use the request object to provide both the required and optional parameters\. 

1. Execute the appropriate method provided by the client by passing in the request object that you created in the preceding step\. 

   The `AmazonDynamoDBClient` client provides `PutItem`, `GetItem`, `UpdateItem`, and `DeleteItem` methods for the CRUD operations\.

## Putting an Item<a name="PutItemLowLevelAPIDotNet"></a>

The `PutItem` method uploads an item to a table\. If the item exists, it replaces the entire item\. 

**Note**  
Instead of replacing the entire item, if you want to update only specific attributes, you can use the `UpdateItem` method\. For more information, see [Updating an Item](#UpdateItemLowLevelDotNet)\.

The following are the steps to upload an item using the low\-level \.NET SDK API\. 

1. Create an instance of the `AmazonDynamoDBClient` class\.

1. Provide the required parameters by creating an instance of the `PutItemRequest` class\.

   To put an item, you must provide the table name and the item\. 

1. Execute the `PutItem` method by providing the `PutItemRequest` object that you created in the preceding step\.

The following C\# code snippet demonstrates the preceding steps\. The example uploads an item to the ProductCatalog table\. 

**Example**  

```
AmazonDynamoDBClient client = new AmazonDynamoDBClient();
string tableName = "ProductCatalog";

var request = new PutItemRequest
{
   TableName = tableName,
   Item = new Dictionary<string, AttributeValue>()
      {
          { "Id", new AttributeValue { N = "201" }},
          { "Title", new AttributeValue { S = "Book 201 Title" }},
          { "ISBN", new AttributeValue { S = "11-11-11-11" }},
          { "Price", new AttributeValue { S = "20.00" }},
          { 
            "Authors", 
            new AttributeValue 
            { SS = new List<string>{"Author1", "Author2"}   }
          }
      }
};
client.PutItem(request);
```

In the preceding example, you upload a book item that has the Id, Title, ISBN, and Authors attributes\. Note that Id is a numeric type attribute and all other attributes are of the string type\. Authors is a String set\. 

### Specifying Optional Parameters<a name="PutItemLowLevelAPIDotNetOptions"></a>

You can also provide optional parameters using the `PutItemRequest` object as shown in the following C\# code snippet\. The sample specifies the following optional parameters:

+ `ExpressionAttributeNames`, `ExpressionAttributeValues`, and `ConditionExpression` specify that the item can be replaced only if the existing item has the ISBN attribute with a specific value\. 

+  `ReturnValues` parameter to request the old item in the response\.

**Example**  

```
var request = new PutItemRequest
 {
   TableName = tableName,
   Item = new Dictionary<string, AttributeValue>()
               {
                   { "Id", new AttributeValue { N = "104" }},
                   { "Title", new AttributeValue { S = "Book 104  Title" }},
                   { "ISBN", new AttributeValue { S = "444-4444444444" }},
                   { "Authors", 
                     new AttributeValue { SS = new List<string>{"Author3"}}}
               },
    // Optional parameters.
    ExpressionAttributeNames = new Dictionary<string,string>()
    {
        {"#I", "ISBN"}
    },
    ExpressionAttributeValues = new Dictionary<string, AttributeValue>()
    {
        {":isbn",new AttributeValue {S = "444-4444444444"}}
    },
    ConditionExpression = "#I = :isbn"

};
var response = client.PutItem(request);
```

For more information, see [PutItem](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_PutItem.html)\.

## Getting an Item<a name="GetItemLowLevelDotNET"></a>

The `GetItem` method retrieves an item\. 

**Note**  
To retrieve multiple items you can use the `BatchGetItem` method\. For more information, see [Batch Get: Getting Multiple Items](#BatchGetLowLevelDotNet)\.

The following are the steps to retrieve an existing item using the low\-level \.NET SDK API\. 

1. Create an instance of the `AmazonDynamoDBClient` class\.

1. Provide the required parameters by creating an instance of the `GetItemRequest` class\.

   To get an item, you must provide the table name and primary key of the item\. 

1. Execute the `GetItem` method by providing the `GetItemRequest` object that you created in the preceding step\.

The following C\# code snippet demonstrates the preceding steps\. The example retrieves an item from the ProductCatalog table\. 

```
AmazonDynamoDBClient client = new AmazonDynamoDBClient();
string tableName = "ProductCatalog";

var request = new GetItemRequest
 {
   TableName = tableName,
   Key = new Dictionary<string,AttributeValue>() { { "Id", new AttributeValue { N = "202" } } },
 };
 var response = client.GetItem(request);

// Check the response.
var result = response.GetItemResult;
var attributeMap = result.Item; // Attribute list in the response.
```

### Specifying Optional Parameters<a name="GetItemLowLevelDotNETOptions"></a>

You can also provide optional parameters using the `GetItemRequest` object as shown in the following C\# code snippet\. The sample specifies the following optional parameters:

+  `ProjectionExpression` parameter to specify the attributes to retrieve\.

+  `ConsistentRead` parameter to perform a strongly consistent read\. To learn more read consistency, see [Read Consistency](HowItWorks.ReadConsistency.md)\.

**Example**  

```
AmazonDynamoDBClient client = new AmazonDynamoDBClient();
string tableName = "ProductCatalog";

var request = new GetItemRequest
 {
   TableName = tableName,
   Key = new Dictionary<string,AttributeValue>() { { "Id", new AttributeValue { N = "202" } } },
   // Optional parameters.
   ProjectionExpression = "Id, ISBN, Title, Authors",
   ConsistentRead = true
 };

 var response = client.GetItem(request);

// Check the response.
var result = response.GetItemResult;
var attributeMap = result.Item;
```

For more information, see [GetItem](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_GetItem.html)\. 

## Updating an Item<a name="UpdateItemLowLevelDotNet"></a>

The `UpdateItem` method updates an existing item if it is present\. You can use the `UpdateItem` operation to update existing attribute values, add new attributes, or delete attributes from the existing collection\. If the item that has the specified primary key is not found, it adds a new item\.

The `UpdateItem` operation uses the following guidelines:

+ If the item does not exist, `UpdateItem` adds a new item using the primary key that is specified in the input\.

+ If the item exists, `UpdateItem` applies the updates as follows:

  + Replaces the existing attribute values by the values in the update

  + If the attribute that you provide in the input does not exist, it adds a new attribute to the item\.

  + If the input attribute is null, it deletes the attribute, if it is present\. 

  + If you use `ADD` for the Action, you can add values to an existing set \(string or number set\), or mathematically add \(use a positive number\) or subtract \(use a negative number\) from the existing numeric attribute value\.

**Note**  
The `PutItem` operation also can perform an update\. For more information, see [Putting an Item](#PutItemLowLevelAPIDotNet)\. For example, if you call `PutItem` to upload an item and the primary key exists, the `PutItem` operation replaces the entire item\. Note that, if there are attributes in the existing item and those attributes are not specified in the input, the `PutItem` operation deletes those attributes\. However, `UpdateItem` only updates the specified input attributes, any other existing attributes of that item remain unchanged\. 

The following are the steps to update an existing item using the low\-level \.NET SDK API\.

1. Create an instance of the `AmazonDynamoDBClient` class\.

1. Provide the required parameters by creating an instance of the `UpdateItemRequest` class\.

   This is the request object in which you describe all the updates, such as add attributes, update existing attributes, or delete attributes\. To delete an existing attribute, specify the attribute name with null value\. 

1. Execute the `UpdateItem` method by providing the `UpdateItemRequest` object that you created in the preceding step\. 

The following C\# code snippet demonstrates the preceding steps\. The example updates a book item in the ProductCatalog table\. It adds a new author to the Authors collection, and deletes the existing ISBN attribute\. It also reduces the price by one\.

```
AmazonDynamoDBClient client = new AmazonDynamoDBClient();
string tableName = "ProductCatalog";

var request = new UpdateItemRequest
{
    TableName = tableName,
    Key = new Dictionary<string,AttributeValue>() { { "Id", new AttributeValue { N = "202" } } },
    ExpressionAttributeNames = new Dictionary<string,string>()
    {
        {"#A", "Authors"},
        {"#P", "Price"},
        {"#NA", "NewAttribute"},
        {"#I", "ISBN"}
    },
    ExpressionAttributeValues = new Dictionary<string, AttributeValue>()
    {
        {":auth",new AttributeValue { SS = {"Author YY","Author ZZ"}}},
        {":p",new AttributeValue {N = "1"}},
        {":newattr",new AttributeValue {S = "someValue"}},
    },

    // This expression does the following:
    // 1) Adds two new authors to the list
    // 2) Reduces the price
    // 3) Adds a new attribute to the item
    // 4) Removes the ISBN attribute from the item
    UpdateExpression = "ADD #A :auth SET #P = #P - :p, #NA = :newattr REMOVE #I"
};
var response = client.UpdateItem(request);
```

### Specifying Optional Parameters<a name="UpdateItemLowLevelDotNETOptions"></a>

You can also provide optional parameters using the `UpdateItemRequest` object as shown in the following C\# code snippet\. It specifies the following optional parameters:

+ `ExpressionAttributeValues` and `ConditionExpression` to specify that the price can be updated only if the existing price is 20\.00\.

+ `ReturnValues` parameter to request the updated item in the response\. 

**Example**  

```
AmazonDynamoDBClient client = new AmazonDynamoDBClient();
string tableName = "ProductCatalog";

var request = new UpdateItemRequest
{
    Key = new Dictionary<string,AttributeValue>() { { "Id", new AttributeValue { N = "202" } } },

    // Update price only if the current price is 20.00.
    ExpressionAttributeNames = new Dictionary<string,string>()
    {
        {"#P", "Price"}
    },
    ExpressionAttributeValues = new Dictionary<string, AttributeValue>()
    {
        {":newprice",new AttributeValue {N = "22"}},
        {":currprice",new AttributeValue {N = "20"}}
    },
    UpdateExpression = "SET #P = :newprice",
    ConditionExpression = "#P = :currprice",
    TableName = tableName,
    ReturnValues = "ALL_NEW" // Return all the attributes of the updated item.
};

var response = client.UpdateItem(request);
```

For more information, see [UpdateItem](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_UpdateItem.html)\. 

## Atomic Counter<a name="AtomicCounterLowLevelDotNet"></a>

You can use `updateItem` to implement an atomic counter, where you increment or decrement the value of an existing attribute without interfering with other write requests\. To update an atomic counter, use `updateItem` with an attribute of type Number in the *UpdateExpression* parameter, and `ADD` as the *Action*\.

The following code snippet demonstrates this, incrementing the *Quantity* attribute by one\.

```
AmazonDynamoDBClient client = new AmazonDynamoDBClient();
string tableName = "ProductCatalog";

var request = new UpdateItemRequest
{
    Key = new Dictionary<string, AttributeValue>() { { "Id", new AttributeValue { N = "121" } } },
    ExpressionAttributeNames = new Dictionary<string, string>()
    {
        {"#Q", "Quantity"}
    },
    ExpressionAttributeValues = new Dictionary<string, AttributeValue>()
    {
        {":incr",new AttributeValue {N = "1"}}
    },
    UpdateExpression = "SET #Q = #Q + :incr",
    TableName = tableName
};

var response = client.UpdateItem(request);
```

## Deleting an Item<a name="DeleteMidLevelDotNet"></a>

The `DeleteItem` method deletes an item from a table\. 

The following are the steps to delete an item using the low\-level \.NET SDK API\. 

1. Create an instance of the `AmazonDynamoDBClient` class\.

1. Provide the required parameters by creating an instance of the `DeleteItemRequest` class\.

    To delete an item, the table name and item's primary key are required\. 

1. Execute the `DeleteItem` method by providing the `DeleteItemRequest` object that you created in the preceding step\. 

**Example**  

```
AmazonDynamoDBClient client = new AmazonDynamoDBClient();
string tableName = "ProductCatalog";

var request = new DeleteItemRequest
{
    TableName = tableName,
    Key = new Dictionary<string,AttributeValue>() { { "Id", new AttributeValue { N = "201" } } },
};

var response = client.DeleteItem(request);
```

### Specifying Optional Parameters<a name="DeleteItemLowLevelDotNETOptions"></a>

You can also provide optional parameters using the `DeleteItemRequest` object as shown in the following C\# code snippet\. It specifies the following optional parameters:

+ `ExpressionAttributeValues` and `ConditionExpression` to specify that the book item can be deleted only if it is no longer in publication \(the InPublication attribute value is false\)\. 

+ `ReturnValues` parameter to request the deleted item in the response\.

**Example**  

```
var request = new DeleteItemRequest
{
    TableName = tableName,
    Key = new Dictionary<string,AttributeValue>() { { "Id", new AttributeValue { N = "201" } } },

    // Optional parameters.
    ReturnValues = "ALL_OLD",
    ExpressionAttributeNames = new Dictionary<string, string>()
    {
        {"#IP", "InPublication"}
    },
    ExpressionAttributeValues = new Dictionary<string, AttributeValue>()
    {
        {":inpub",new AttributeValue {BOOL = false}}
    },
    ConditionExpression = "#IP = :inpub"
};

var response = client.DeleteItem(request);
```

For more information, see [DeleteItem](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_DeleteItem.html)\.

## Batch Write: Putting and Deleting Multiple Items<a name="BatchWriteLowLevelDotNet"></a>

Batch write refers to putting and deleting multiple items in a batch\. The `BatchWriteItem` method enables you to put and delete multiple items from one or more tables in a single call\. The following are the steps to retrieve multiple items using the low\-level \.NET SDK API\.

1. Create an instance of the `AmazonDynamoDBClient` class\.

1. Describe all the put and delete operations by creating an instance of the `BatchWriteItemRequest` class\.

1. Execute the `BatchWriteItem` method by providing the `BatchWriteItemRequest` object that you created in the preceding step\.

1. Process the response\. You should check if there were any unprocessed request items returned in the response\. This could happen if you reach the provisioned throughput limit or some other transient error\. Also, DynamoDB limits the request size and the number of operations you can specify in a request\. If you exceed these limits, DynamoDB rejects the request\. For more information, see [BatchWriteItem](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_BatchWriteItem.html)\. 

The following C\# code snippet demonstrates the preceding steps\. The example creates a `BatchWriteItemRequest` to perform the following write operations:

+ Put an item in Forum table

+ Put and delete an item from Thread table

The code then executes `BatchWriteItem` to perform a batch operation\.

```
AmazonDynamoDBClient client = new AmazonDynamoDBClient();

string table1Name = "Forum";
string table2Name = "Thread";

var request = new BatchWriteItemRequest
 {
   RequestItems = new Dictionary<string, List<WriteRequest>>
    {
      {
        table1Name, new List<WriteRequest>
        {
          new WriteRequest
          {
             PutRequest = new PutRequest
             {
                Item = new Dictionary<string,AttributeValue>
                {
                  { "Name", new AttributeValue { S = "Amazon S3 forum" } },
                  { "Threads", new AttributeValue { N = "0" }}
                }
             }
          } 
        }
      } ,
      {
        table2Name, new List<WriteRequest>
        {
          new WriteRequest
          {
            PutRequest = new PutRequest
            {
               Item = new Dictionary<string,AttributeValue>
               {
                 { "ForumName", new AttributeValue { S = "Amazon S3 forum" } },
                 { "Subject", new AttributeValue { S = "My sample question" } },
                 { "Message", new AttributeValue { S = "Message Text." } },
                 { "KeywordTags", new AttributeValue { SS = new List<string> { "Amazon S3", "Bucket" }  } }
               }
            }
          },
          new WriteRequest
          {
             DeleteRequest = new DeleteRequest
             {
                Key = new Dictionary<string,AttributeValue>()
                {
                   { "ForumName", new AttributeValue { S = "Some forum name" } },
                   { "Subject", new AttributeValue { S = "Some subject" } }
                }
             }
          }
        }
      } 
    }
 };
response = client.BatchWriteItem(request);
```

For a working example, see [Example: Batch Operations Using AWS SDK for \.NET Low\-Level API](batch-operation-lowlevel-dotnet.md)\. 

## Batch Get: Getting Multiple Items<a name="BatchGetLowLevelDotNet"></a>

The `BatchGetItem` method enables you to retrieve multiple items from one or more tables\. 

**Note**  
To retrieve a single item you can use the `GetItem` method\. 

The following are the steps to retrieve multiple items using the low\-level \.NET SDK API\.

1. Create an instance of the `AmazonDynamoDBClient` class\.

1. Provide the required parameters by creating an instance of the `BatchGetItemRequest` class\.

   To retrieve multiple items, the table name and a list of primary key values are required\. 

1. Execute the `BatchGetItem` method by providing the `BatchGetItemRequest` object that you created in the preceding step\.

1. Process the response\. You should check if there were any unprocessed keys, which could happen if you reach the provisioned throughput limit or some other transient error\.

The following C\# code snippet demonstrates the preceding steps\. The example retrieves items from two tables, Forum and Thread\. The request specifies two items in the Forum and three items in the Thread table\. The response includes items from both of the tables\. The code shows how you can process the response\.

```
AmazonDynamoDBClient client = new AmazonDynamoDBClient();

string table1Name = "Forum";
string table2Name = "Thread";

var request = new BatchGetItemRequest
{
  RequestItems = new Dictionary<string, KeysAndAttributes>()
  {
    { table1Name, 
      new KeysAndAttributes
      {
        Keys = new List<Dictionary<string, AttributeValue>>()
        {
          new Dictionary<string, AttributeValue>()
          {
            { "Name", new AttributeValue { S = "DynamoDB" } }
          },
          new Dictionary<string, AttributeValue>()
          {
            { "Name", new AttributeValue { S = "Amazon S3" } }
          }
        }
      }
    },
    {
      table2Name,
      new KeysAndAttributes
      {
        Keys = new List<Dictionary<string, AttributeValue>>()
        {
          new Dictionary<string, AttributeValue>()
          {
            { "ForumName", new AttributeValue { S = "DynamoDB" } },
            { "Subject", new AttributeValue { S = "DynamoDB Thread 1" } }
          },
          new Dictionary<string, AttributeValue>()
          {
            { "ForumName", new AttributeValue { S = "DynamoDB" } },
            { "Subject", new AttributeValue { S = "DynamoDB Thread 2" } }
          },
          new Dictionary<string, AttributeValue>()
          {
            { "ForumName", new AttributeValue { S = "Amazon S3" } },
            { "Subject", new AttributeValue { S = "Amazon S3 Thread 1" } }
          }
        }
      }
    }
  }
};

var response = client.BatchGetItem(request);

// Check the response.
var result = response.BatchGetItemResult;
var responses = result.Responses; // The attribute list in the response.

var table1Results = responses[table1Name];
Console.WriteLine("Items in table {0}" + table1Name);
foreach (var item1 in table1Results.Items)
{
  PrintItem(item1);
}

var table2Results = responses[table2Name];
Console.WriteLine("Items in table {1}" + table2Name);
foreach (var item2 in table2Results.Items)
{
  PrintItem(item2);
}
// Any unprocessed keys? could happen if you exceed ProvisionedThroughput or some other error.
Dictionary<string, KeysAndAttributes> unprocessedKeys = result.UnprocessedKeys;
foreach (KeyValuePair<string, KeysAndAttributes> pair in unprocessedKeys)
{
    Console.WriteLine(pair.Key, pair.Value);
}
```

### Specifying Optional Parameters<a name="BatchGetItemLowLevelDotNETOptions"></a>

You can also provide optional parameters using the `BatchGetItemRequest` object as shown in the following C\# code snippet\. The code samples retrieves two items from the Forum table\. It specifies the following optional parameter:

+  `ProjectionExpression` parameter to specify the attributes to retrieve\.

**Example**  

```
AmazonDynamoDBClient client = new AmazonDynamoDBClient();

string table1Name = "Forum";

var request = new BatchGetItemRequest
{
  RequestItems = new Dictionary<string, KeysAndAttributes>()
  {
    { table1Name, 
      new KeysAndAttributes
      {
        Keys = new List<Dictionary<string, AttributeValue>>()
        {
          new Dictionary<string, AttributeValue>()
          {
            { "Name", new AttributeValue { S = "DynamoDB" } }
          },
          new Dictionary<string, AttributeValue>()
          {
            { "Name", new AttributeValue { S = "Amazon S3" } }
          }
        }
      },
      // Optional - name of an attribute to retrieve.
      ProjectionExpression = "Title"
    }
  }
};

var response = client.BatchGetItem(request);
```

For more information, see [BatchGetItem](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_BatchGetItem.html)\. 