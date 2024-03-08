# Calculating the size of a DynamoDB item

## Background

According to the [AWS Docs](https://docs.aws.amazon.com/dynamodb/), DynamoDB is:

> [...] a fully managed NoSQL database service that provides fast and predictable performance with seamless scalability. You can use Amazon DynamoDB to create a database table that can store and retrieve any amount of data, and serve any level of request traffic. Amazon DynamoDB automatically spreads the data and traffic for the table over a sufficient number of servers to handle the request capacity specified by the customer and the amount of data stored, while maintaining consistent and fast performance.

From having worked with DynamoDB it is clear that it is, like many other databases, a powerful tool when used correctly. Surely, there are already a shit-ton of blog posts and articles on best practices and practical tips. Some of my learnings might be a candidate for a future post, but this one is about a very specific topic: how to calculate the size of a DynamoDB item.

There are already a couple tools that do this, I will list the ones I've seen:

- Zac Charles's [DynamoDB Item Size and Consumed Capacity Calculator](https://zaccharles.github.io/dynamodb-calculator/): a web UI for copy-pasting your JSON or DynamoDB JSON. The drawback is the fact that this is web-based: you need to copy-paste your JSON/Dynamo JSON object. From checking the code, the calculation seems to run client-side so at least your object is not uploaded anywhere, but I have a tendency to not do this, especially when doing work-related tasks.
  - [Github](https://github.com/zaccharles/dynamodb-calculator)
  - [Blog post](https://zaccharles.medium.com/calculating-a-dynamodb-items-size-and-consumed-capacity-d1728942eb7c)
- Leeroy Hannigan's DynamoDB-ItemSizeCalculator: a calculator written in JavaScript. The drawback here is that it's written in JavaScript, which is not what I'm using currently.
  - [Github](https://github.com/LeeroyHannigan/DynamoDB-ItemSizeCalculator)
- yu-yamanoi's [dynamodb-item-size-calculator](https://github.com/yyamanoi1222/dynamodb-item-size-calculator): a calculator written in Go based on parsing JSON or Dynamo JSON. The first drawback is that this first and foremost is a CLI. The second is that this package partly reimplements the marshalling logic. I'd rather use the official AWS SDK for Go, as we will see below. The examples are also using the first version of the SDK for Go. As of writing, the latest version is [v2](https://github.com/aws/aws-sdk-go-v2).

Thanks to the authors/contributors of the above listed tools for the inspiration!

## Problem & Solution

It's simple. I'm mostly writing Go and would like there to be a way to calculate the size of a DynamoDB item before attempting to write the item.

> ⚠️ Coming back to best practices: If you have defined your schema in a way that your objects run the risk of approaching the size limit, you should probably consider a different approach. Anyway, this is about size calculation of items, not best practices.

The solution is to use the `dynamodb/attributevalue` package from the `aws-sdk-g-v2` library. The `MarshalMap` function from this package will recursively marshal a Go struct into a DynamoDB attribute value map. It has the following signature:

```go
func attributevalue.MarshalMap(in interface{}) (map[string]types.AttributeValue, error)
```

> ℹ️ Alternatively, we can use the `Marshal` function to marshal a Go struct into a single `types.AttributeValue`. However, for Go structs this will always return a `types.AttributeValueMemberM` (map) at the top level which does not provide any additional value. Additionally, the `dynamodb` Client has the signature `func (c *Client) PutItem(ctx context.Context, params *PutItemInput, optFns ...func(*Options)) (*PutItemOutput, error)` which takes a `PutItemInput` struct. The `PutItemInput`'s required `Item` field of type `map[string]types.AttributeValue`, we might as well use `MarshalMap` to get the `map[string]types.AttributeValue` directly.

Now, we can recursively traverse the `map[string]types.AttributeValue` and calculate the size of each attribute+value pair. For every particular `AttributeValue` type, we reference the [DynamoDB Item size and formats doc](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/CapacityUnitCalculations.html) to calculate the size of the item. Below is some of the relevant information from the doc and some pseudo code for the calculation:

- Strings (`types.AttributeValueMemberS`):

  > _- "Strings are Unicode with UTF-8 binary encoding. The size of a string is (number of UTF-8-encoded bytes of attribute name) + (number of UTF-8-encoded bytes)."_

  Strings in Go are UTF-8 encoded, so we can use `len` to get the number of bytes.

  ```go
  size += len(*av.Value)
  ```

- Numbers (`types.AttributeValueMemberN`):

  > _- "Numbers are variable length, with up to 38 significant digits. Leading and trailing zeroes are trimmed. The size of a number is approximately (number of UTF-8-encoded bytes of attribute name) + (1 byte per two significant digits) + (1 byte)."_

  We can use the `len` function to get actual number of bytes of the string representation of the number.

  ```go
  size += len(*av.Value)
  ```

- Binary (`types.AttributeValueMemberB`):

  > _- "A binary value must be encoded in base64 format before it can be sent to DynamoDB, but the value's raw byte length is used for calculating size. The size of a binary attribute is (number of UTF-8-encoded bytes of attribute name) + (number of raw bytes)."_

  We can use `len` to get the number of bytes of the encoded binary value.

  ```go
    size += len(av.Value)
  ```

- Boolean (`types.AttributeValueMemberBOOL`) and Null (`types.AttributeValueMemberNULL`):

  > _- "The size of a null attribute or a Boolean attribute is (number of UTF-8-encoded bytes of attribute name) + (1 byte)."_

  ```go
  size += 1
  ```

- Lists (`types.AttributeValueMemberL`) and Maps (`types.AttributeValueMemberM`):

  > _- "An attribute of type List or Map requires 3 bytes of overhead, regardless of its contents. The size of a List or Map is (number of UTF-8-encoded bytes of attribute name) + sum (size of nested elements) + (3 bytes). The size of an empty List or Map is (number of UTF-8-encoded bytes of attribute name) + (3 bytes)."_ > \
  > _- "Each List or Map element also requires 1 byte of overhead."_

  ```go
  // list
  size += 3
  for _, v := range av.Value {
    size += calculateSize(v)
    size++
  }

  // map
  size += 3
  for k, v := range av.Value {
    size += len(k)
    size += calculateSize(v)
    size++
  }
  ```

For the actual implementation, see the [github.com/ryeguard/ddbcalc](https://github.com/ryeguard/ddbcalc) repo.

## Details

When writing the tests in the repo, I realized that the Dynamo Set types, i.e., `types.AttributeValueMemberSS`, `types.AttributeValueMemberNS`, and `types.AttributeValueMemberBS`, will not be used by the `MarshalMap` function by default. This follows quite naturally because Go does not have a built-in set type (unless you are using a `map` as a kind of set).

In Dynamo, set type can represent multiple scalar values of type string, number, or binary. The properties of a set differ from those of a list according to [Supported data types and naming rules in Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.NamingRulesDataTypes.html) as follows:

- All the elements within a set must be of the same type.
- Each element within a set must be unique.
- The order of the values within a set is not preserved.
- Sets cannot be empty.

In other words, if we want to utilize the set types while using Go, we will have to tag our struct field in question with the `dynamodbav:",stringset"`, `dynamodbav:",numberset"`, or `dynamodbav:",binaryset"` tag like so:

```go
type Item struct {
  MyStringSetField []string `dynamodbav:",stringset"`
  MyNumberSetField []string `dynamodbav:",numberset"`
  MyBinarySetField []string `dynamodbav:"myField,binaryset"` // also changing the field name to "myField"
}
```

Since sets are not an inherent part of Go, it is arguably more natural to use lists instead. However, it is good to be aware of the possibility of using sets as it might work really well for some use cases.

Resources I found useful:

- [APIReference: AttributeValue](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_AttributeValue.html)
- [Using DynamoDB With Golang](https://thomasstep.com/blog/using-dynamodb-with-golang)
- [DynamoDB Item Size and Consumed Capacity Calculator](https://zaccharles.github.io/dynamodb-calculator/)
