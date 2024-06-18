# 第七章 操控数据

每个应用程序都使用数据，并且我们需要将数据从一种形式转换为另一种形式。本章提供了关于数据转换的多个主题，如机密管理、JSON 序列化和 XML 序列化。

机密信息是我们不希望向第三方公开的数据，例如密码或 API 密钥。本章包括三节关于管理这些机密信息的内容，包括哈希处理、加密和隐藏存储。

当今我们处理的许多数据都是以 JSON 格式。在现代框架中，基本的序列化/反序列化操作很简单，如果你同时拥有数据的消费者和提供者，这一切会更加简单。但是当你处理第三方数据时，你无法控制数据的一致性或标准。因此，本章的 JSON 部分深入探讨了定制方法，帮助你处理任何你需要的 JSON 格式数据。

最后，尽管 JSON 在当前互联网数据格式中占据主导地位，但仍然有大量 XML 数据需要处理，这是 XML 章节的主题。你将看到 LINQ 的另一种风格，称为 LINQ to XML，它可以完全控制序列化/反序列化过程。

# 7.1 生成密码哈希

## 问题

你需要安全地存储用户密码。

## 解决方案

这种方法生成一个随机盐来保护秘密：

```cs
static byte[] GenerateSalt()
{
    const int SaltLength = 64;

    byte[] salt = new byte[SaltLength];
    var rngRand = new RNGCryptoServiceProvider();

    rngRand.GetBytes(salt);

    return salt;
}
```

接下来的两种方法使用该盐生成哈希值：

```cs
static byte[] GenerateMD5Hash(string password, byte[] salt)
{
    byte[] passwordBytes = Encoding.UTF8.GetBytes(password);

    byte[] saltedPassword =
        new byte[salt.Length + passwordBytes.Length];

    using var hash = new MD5CryptoServiceProvider();

    return hash.ComputeHash(saltedPassword);
}

static byte[] GenerateSha256Hash(string password, byte[] salt)
{
    byte[] passwordBytes = Encoding.UTF8.GetBytes(password);

    byte[] saltedPassword =
        new byte[salt.Length + passwordBytes.Length];

    using var hash = new SHA256CryptoServiceProvider();

    return hash.ComputeHash(saltedPassword);
}
```

这里是如何使用方法生成哈希值：

```cs
static void Main(string[] args)
{
    Console.WriteLine("\nPassword Hash Demo\n");

    Console.Write("What is your password? ");
    string password = Console.ReadLine();

    byte[] salt = GenerateSalt();

    byte[] md5Hash = GenerateMD5Hash(password, salt);
    string md5HashString = Convert.ToBase64String(md5Hash);
    Console.WriteLine($"\nMD5:    {md5HashString}");

    byte[] sha256Hash = GenerateSha256Hash(password, salt);
    string sha256HashString = Convert.ToBase64String(sha256Hash);
    Console.WriteLine($"\nSHA256: {sha256HashString}");
}
```

## 讨论

ASP.NET Identity 对密码和组/角色管理提供了很好的支持，这应该是在规划新项目时考虑的要点之一。然而，在某些情况下，例如当你必须使用 ASP.NET Identity 不支持的数据库或必须使用具有自己自制密码管理系统的现有数据库时，ASP.NET Identity 可能不是最佳选择。

在构建自定义密码管理解决方案时，最佳实践是使用盐对密码进行哈希处理。哈希是将密码单向转换为一串无法理解的字符。每次你对特定密码进行哈希处理时，都会得到相同的哈希值。然而，与加密的重要区别在于你无法解密哈希值——没有办法将哈希值翻译回原始密码。这就引出了一个问题：如何知道用户输入了正确的密码。由于本书主要讲解 C#，数据库开发超出了本书的范围。尽管如此，在这里我们列出了验证密码所需的步骤：

1.  创建用户账户时，对密码进行哈希处理，并将哈希值与用户名一起存储在数据库中。

1.  当用户登录时，他们提供用户名和密码。

1.  使用用户名，你的代码进行数据库查询，并检索匹配的哈希值。

1.  对用户输入的密码进行哈希处理。

1.  比较哈希密码。

1.  如果密码哈希匹配，则验证成功——否则验证失败。

安全性是一场持续的猫鼠游戏。我们刚刚学会用哈希保护密码，黑客们就寻找方法突破它。最终，我们能做的最好的事情就是找到一定程度的安全性，使得黑客因我们需要保护信息而愿意获取它的成本变得非常高昂。您能承受多少安全性成本？

幸运的是，有一个简单的方法来加强密码安全性。在处理哈希密码的安全最佳实践中，包含盐（salt）是一种方法，盐是追加到密码后的随机字节数组。我们将盐和用户名、密码一起保存在数据库中。这对于防范彩虹表攻击非常有效，详见注释。解决方案中的`GenerateSalt`方法生成一个随机的 64 字节值。盐可以防止彩虹表攻击，并迫使黑客转向更为计算密集的字典攻击。

###### 注释

如果黑客侵入您的系统或者找到了存储密码的表格的副本，那么常见的攻击有两种：字典攻击和彩虹表攻击。

在字典攻击中，黑客拥有一个包含单词和短语的字典列表，并逐个对列表中的项进行哈希处理，然后与数据库表进行比较。尽管有所有的复杂规则和遵循这些规则的人数，总会有些人使用单个单词密码。剧透警告：对于那些认为用符号/数字字符替换会变得聪明的人，这种方法行不通；黑客的字典和算法已经考虑到了这一点。

彩虹表攻击是字典攻击的另一种变体，其区别在于彩虹表已经对常见单词进行了哈希处理，因此他们只需进行简单的比较即可快速地遍历密码表。

`GenerateMD5Hash`和`GenerateSha256`哈希方法都接受一个密码和一个盐。这两种方法都将密码转换为`byte[]`，连接密码和盐，然后生成一个哈希。MD5 和 SHA256 实现之间的语法差异在于`MD5CryptoServiceProvider`与`SHA256​Cryp⁠to​ServiceProvider`。

在实践中，有不同的原因使用特定的哈希算法。.NET 框架有几种哈希算法，你可以通过查找`HashAlgorithm`并检查其派生类来找到这些算法。许多最近的实现使用 SHA256 哈希，因为它比早期的哈希算法提供更好的保护。我包括 MD5 算法是为了说明你并不总是能够选择算法，因为密码表可能已经使用 MD5 创建。在这种情况下，对用户的不便可能会阻止他们重新输入密码以适应另一种哈希算法。

`Main` 方法演示如何使用这些算法生成哈希值。这里的一个有趣之处是调用 `Convert.ToBase64String`。每当你在不同地方传输数据时，传输机制都有一套基于特殊字符的协议和格式。如果哈希字节中的字符在传输过程中转换为特殊字符，软件将会出现问题。解决这个问题的标准方法是使用一种名为 Base64 的数据格式，它生成的字符不会与特殊的数据格式或传输协议字符冲突。

# 7.2 加密和解密机密信息

## 问题

你有需要在静态状态下加密的 API 密钥。

## 解决方案

这个类用于加密和解密机密信息：

```cs
public class Crypto
{
    public byte[] Encrypt(string plainText, byte[] key)
    {
        using Aes aes = Aes.Create();
        aes.Key = key;

        using var memStream = new MemoryStream();
        memStream.Write(aes.IV, 0, aes.IV.Length);

        using var cryptoStream = new CryptoStream(
            memStream,
            aes.CreateEncryptor(),
            CryptoStreamMode.Write);

        byte[] plainTextBytes = Encoding.UTF8.GetBytes(plainText);

        cryptoStream.Write(plainTextBytes);
        cryptoStream.FlushFinalBlock();

        memStream.Position = 0;

        return memStream.ToArray();
    }

    public string Decrypt(byte[] cypherBytes, byte[] key)
    {
        using var memStream = new MemoryStream();
        memStream.Write(cypherBytes);
        memStream.Position = 0;

        using var aes = Aes.Create();

        byte[] iv = new byte[aes.IV.Length];
        memStream.Read(iv, 0, iv.Length);

        using var cryptoStream = new CryptoStream(
            memStream,
            aes.CreateDecryptor(key, iv),
            CryptoStreamMode.Read);

        int plainTextByteLength = cypherBytes.Length - iv.Length;
        var plainTextBytes = new byte[plainTextByteLength];
        cryptoStream.Read(plainTextBytes, 0, plainTextByteLength);

        return Encoding.UTF8.GetString(plainTextBytes);
    }
}
```

这是一个生成随机密钥的方法：

```cs
static byte[] GenerateKey()
{
    const int KeyLength = 32;

    byte[] key = new byte[KeyLength];
    var rngRand = new RNGCryptoServiceProvider();

    rngRand.GetBytes(key);

    return key;
}
```

使用 `Crypto` 类和随机密钥加密和解密机密信息的方法如下：

```cs
static void Main()
{
    var crypto = new Crypto();

    Console.Write("Please enter text to encrypt: ");
    string userPlainText = Console.ReadLine();

    byte[] key = GenerateKey();

    byte[] cypherBytes = crypto.Encrypt(userPlainText, key);

    string cypherText = Convert.ToBase64String(cypherBytes);

    Console.WriteLine($"Cypher Text: {cypherText}");

    string decryptedPlainText = crypto.Decrypt(cypherBytes, key);

    Console.WriteLine($"Plain Text: {decryptedPlainText}");
}
```

## 讨论

我们经常有需要保护的秘密信息——API 密钥或其他敏感信息。加密是在静止状态下保护信息的方法。在保存之前，我们对数据进行加密，然后在检索加密数据后，我们对其进行解密以供使用。

在解决方案中，`Crypto` 类具有加密和解密数据的方法。`key` 参数是加密/解密算法使用的秘密值。我们将使用称为 *对称密钥加密* 的技术，其中我们使用单个密钥来加密/解密所有数据。显然，你不应将加密密钥存储在与数据相同的位置，因为如果黑客能够读取数据，他们还需要找出加密密钥所在的位置。在此演示中，`GenerateKey` 方法生成一个随机的 32 位密钥，加密提供程序所需。

加密提供程序是使用特殊算法加密/解密数据的代码。解决方案示例使用了先进加密标准 (AES)，这是一种现代和安全的加密算法。

在保存数据时，你将 `plainText` 字符串与 `key` 一起传递到 `Encrypt` 方法中。调用 `AES.Create` 返回 `AES` 的一个实例。存储在数据库中的值是连接的初始化向量 (IV) 和加密文本。注意 `memStream` 首先从 `AES` 实例加载 IV 的方式。

`CryptoStream` 的三个参数是 `memStream`（包含 IV）、`ICryptoTransform`（通过调用 `AES.CreateEncryptor` 返回）、以及 `Crypto​S⁠treamMode`（指示我们正在向流中写入）。`cryptoStream` 实例将加密后的字节追加到 `memStream` 中的 IV。我们使用数据的 `byte[]` 表示，包括 `plainText`。在 `cryptoStream` 上调用 `Write` 执行加密，在调用 `FlushFinalBlock` 确保所有字节都被处理并推送到 `memStream` 中。

`Decrypt` 方法反转了这个过程。除了与加密时相同的 `key` 外，还有一个 `cypherBytes` 参数。如果您回忆一下 `Encrypt` 过程，加密值包括 IV 和附加的加密值，这些是 `cypherBytes` 的内容。加载了 `cypherBytes` 到 `memStream` 后，代码将 `memStream` 重新定位到开头，并将 IV 提取到 `iv` 中。这样 `memStream` 就位于 IV 的长度处，加密值从这里开始。

这里，`cryptoStream` 使用了加密文本（`memStream` 适当位置）。在这里，`ICryptoTransform` 不同，因为我们使用 `iv` 和 `key` 调用 `CreateDecryptor`。此外，`CryptoStreamMode` 需要设置为 `Read`。在 `cryptoStream` 上调用 `Read` 执行解密操作。

`Main` 方法展示了如何使用 `Encrypt` 和 `Decrypt` 方法。请注意，它们都使用相同的 `key`。`Convert.ToBase64String` 确保我们可以处理数据，避免随机字节被意外解释。例如，如果将二进制文件打印到控制台，可能会听到响声，因为某些字节被解释为响铃字符。此外，在传输数据时，Base64 可以避免字节被解释为传输协议或格式字符，从而破坏代码。

# 7.3 隐藏开发秘密

## 问题

您需要避免将密码和 API 密钥等秘密信息意外提交到源代码控制中。

## 解决方案

这是项目文件：

```cs
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net5.0</TargetFramework>
    <RootNamespace>Section_07_03</RootNamespace>
    <UserSecretsId>d3d91a8b-d440-414a-821e-7f11eec48f32</UserSecretsId>
    </PropertyGroup>

    <ItemGroup>
    <PackageReference
        Include="Microsoft.Extensions.Hosting" Version="5.0.0" />
    </ItemGroup>
</Project>
```

以下代码展示了如何轻松添加支持隐藏秘密的代码：

```cs
class Program
{
    static void Main()
    {
        var config = new ConfigurationBuilder()
            .AddUserSecrets<Program>()
            .Build();

        string key = "CSharpCookbook:ApiKey";
        Console.WriteLine($"{key}: {config[key]}");
    }
}
```

## 讨论

这是很常见的问题，开发人员意外将数据库连接字符串从配置文件提交到源代码控制中。另一个常见问题是，开发人员在在线论坛寻求帮助时，在其代码示例中意外留下了秘密。希望能清楚地看到这些错误可能对应用程序甚至业务造成严重损害。

其中一种解决方案是使用 Secret Manager。虽然 Secret Manager 通常与 ASP.NET 关联紧密，因为它具有内置的配置支持，但您可以将其与任何类型的应用程序一起使用。该解决方案展示了如何在控制台应用程序中轻松使用 Secret Manager。

###### 注意

这是在开发环境中非常有用的功能。在生产环境中，您可能希望使用更安全的选项，例如，如果您部署到 Azure，则可能会使用密钥保管库（Key Vault）。将机密保存在环境变量中是避免将其存储在代码或配置中的另一种方式。

有些项目类型，如 ASP.NET，已经支持确保不会意外将开发代码部署到生产环境中，例如：

```cs
if (env.IsDevelopment())
{
    config.AddUserSecrets<Program>();
}
```

您可以使用 dotnet CLI 配置应用程序以使用 Secret Manager。第一步是通过命令行更新项目：

```cs
dotnet user-secrets init
```

这在项目文件中添加了一个`UserSecretsID`标签，如前所示。该 GUID 标识了存储秘密的文件系统位置。在这个示例中，该位置是：

```cs
%APPDATA%\Microsoft\UserSecrets\d3d91a8b-d440-414a-821e-7f11eec48f32
\secrets.json
```

在 Windows 上或：

```cs
~/.microsoft/usersecrets/d3d91a8b-d440-414a-821e-7f11eec48f32
/secrets.json
```

对于 Linux 或 macOS 机器。

设置完成后，您可以开始添加秘密，就像这样（与项目文件夹相同的位置）：

```cs
dotnet user-secrets set "CSharpCookbook:ApiKey" "mYaPIsECRET"
```

您可以通过查看`secrets.json`或以下命令来验证已保存的秘密：

```cs
dotnet user-secrets list
```

`Main`方法展示了如何读取秘密管理器的密钥。记得引用`Microsoft.Extensions.Hosting` NuGet 包。只需在新的`ConfigurationBuilder`上调用`AddUserSecrets`。在其上调用`Build`返回一个`IConfigurationRoot`实例，提供索引器支持以读取键。

# 7.4 生成 JSON

## 问题

你需要自定义 JSON 输出格式。

## 解决方案

此代码显示了`PurchaseOrder`的外观：

```cs
public enum PurchaseOrderStatus
{
    Received,
    Processing,
    Fulfilled
}

public class PurchaseItem
{
 [JsonPropertyName("serialNo")]
    public string SerialNumber { get; set; }

 [JsonPropertyName("description")]
    public string Description { get; set; }

 [JsonPropertyName("qty")]
    public float Quantity { get; set; }

 [JsonPropertyName("amount")]
    public decimal Price { get; set; }
}

public class PurchaseOrder
{
 [JsonPropertyName("company")]
    public string CompanyName { get; set; }
 [JsonPropertyName("address")]
    public string Address { get; set; }
 [JsonPropertyName("phone")]
    public string Phone { get; set; }

 [JsonPropertyName("status")]
    public PurchaseOrderStatus Status { get; set; }

 [JsonPropertyName("other")]
    public Dictionary<string, string> AdditionalInfo { get; set; }

 [JsonPropertyName("details")]
    public List<PurchaseItem> Items { get; set; }
}
```

此代码序列化了`PurchaseOrder`：

```cs
public class PurchaseOrderService
{
    public void View(PurchaseOrder po)
    {
        var jsonOptions = new JsonSerializerOptions
        {
            WriteIndented = true
        };

        string poJson = JsonSerializer.Serialize(po, jsonOptions);

        // send HTTP request

        Console.WriteLine(poJson);
    }
}
```

这是如何填充`PurchaseOrder`的方法：

```cs
static PurchaseOrder GetPurchaseOrder()
{
    return new PurchaseOrder
    {
        CompanyName = "Acme, Inc.",
        Address = "123 4th St.",
        Phone = "555-835-7609",
        AdditionalInfo = new Dictionary<string, string>
        {
            { "terms", "Net 30" },
            { "poc", "J. Smith" }
        },
        Items = new List<PurchaseItem>
        {
            new PurchaseItem
            {
                Description = "Widget",
                Price = 13.95m,
                Quantity = 5,
                SerialNumber = "123"
            }
        }
    };
}
```

`Main`方法驱动该过程：

```cs
static void Main()
{
    PurchaseOrder po = GetPurchaseOrder();
    new PurchaseOrderService().View(po);
}
```

这是输出结果：

```cs
{
  "company": "Acme, Inc.",
  "address": "123 4th St.",
  "phone": "555-835-7609",
  "status": 0,
  "other": {
    "terms": "Net 30",
    "poc": "J. Smith"
  },
  "details": [
    {
      "serialNo": "123",
      "description": "Widget",
      "qty": 5,
      "amount": 13.95
    }
  ]
}
```

## 讨论

只需调用`JsonSerializer.Serialize`，来自`System.Text.Json`命名空间，这是将对象序列化为 JSON 的简单快速方法。如果您拥有应用程序的生产和消费部分，这可能是简单和快速的选择。但通常情况下，您会消费一个指定其自己 JSON 数据格式的第三方 API。此外，其命名约定与 C# Pascal 大小写属性名称不匹配。本节显示如何执行这些序列化器输出的自定义。

###### 注意

Microsoft 在.NET Core 3 中引入了`System.Text.Json`命名空间。此前，一个广受欢迎的选择是得到了出色支持的`Newtonsoft.Json`库。

在解决方案场景中，我们希望将 JSON 文档发送到 API，但属性名称不匹配。这就是为什么`PurchaseOrder`（及其支持类型）使用`JsonPropertyName`属性装饰属性。`JsonSerializer`使用`JsonPropertyName`指定输出属性名称。

`PurchaseOrderService`有一个`View`方法，可以序列化`PurchaseOrder`。默认情况下，序列化器输出是单行的，我们希望看到格式化输出。代码使用了一个`JsonSerializerOption`，其中`WriteIndented`设置为`true`，产生了解决方案中显示的输出。

`Main`方法驱动该过程，获取一个新的`PurchaseOrder`，然后调用`View`打印出结果。

有时，API 会有机地增长，它们的命名约定缺乏一致性，这使得自定义输出的理想方法。但是，如果您使用的是具有一致命名约定的 API，7.5 章解释了如何构建转换器以避免为每个属性都装饰`JsonPropertyName`。

## 参见

7.5 章，“消费 JSON”

# 7.5 消费 JSON

## 问题

您需要读取不符合默认反序列化选项的 JSON 对象。

## 解决方案

这是`PurchaseOrder`的外观：

```cs
public enum PurchaseOrderStatus
{
    Received,
    Processing,
    Fulfilled
}

public class PurchaseItem
{
    public string SerialNumber { get; set; }

    public string Description { get; set; }

    public float Quantity { get; set; }

    public decimal Price { get; set; }
}

public class PurchaseOrder
{
    public string CompanyName { get; set; }
    public string Address { get; set; }
    public string Phone { get; set; }

 [JsonConverter(typeof(PurchaseOrderStatusConverter))]
    public PurchaseOrderStatus Status { get; set; }

    public Dictionary<string, string> AdditionalInfo { get; set; }

    public List<PurchaseItem> Items { get; set; }
}
```

这是一个自定义`JsonConverter`类：

```cs
public class PurchaseOrderStatusConverter
    : JsonConverter<PurchaseOrderStatus>
{
    public override PurchaseOrderStatus Read(
        ref Utf8JsonReader reader,
        Type typeToConvert,
        JsonSerializerOptions options)
    {
        string statusString = reader.GetString();

        if (Enum.TryParse(
            statusString,
            out PurchaseOrderStatus status))
        {
            return status;
        }
        else
        {
            throw new JsonException(
                $"{statusString} is not a valid " +
                $"{nameof(PurchaseOrderStatus)} value.");
        }
    }

    public override void Write(
        Utf8JsonWriter writer,
        PurchaseOrderStatus value,
        JsonSerializerOptions options)
    {
        writer.WriteStringValue(value.ToString());
    }
}
```

这是自定义的 JSON 命名策略：

```cs
public class SnakeCaseNamingPolicy : JsonNamingPolicy
{
    public override string ConvertName(string name)
    {
        var targetChars = new List<char>();
        char[] sourceChars = name.ToCharArray();

        char first = sourceChars[0];
        if (char.IsUpper(first))
            targetChars.Add(char.ToLower(first));
        else
            targetChars.Add(first);

        for (int i = 1; i < sourceChars.Length; i++)
        {
            char ch = sourceChars[i];

            if (char.IsUpper(ch))
            {
                targetChars.Add('_');
                targetChars.Add(char.ToLower(ch));
            }
            else
            {
                targetChars.Add(ch);
            }
        }

        return new string(targetChars.ToArray());
    }
}
```

此类模拟请求，返回格式化为 JSON 的数据：

```cs
public class PurchaseOrderService
{
    public string Get(int poID)
    {
        // get HTTP request

        return @"{
""company_name"": ""Acme, Inc."",
""address"": ""123 4th St."",
""phone"": ""555-835-7609"",
""additional_info"": {
 ""terms"": ""Net 30"",
 ""poc"": ""J. Smith"",
},
""status"": ""Processing"",
""items"": [
 {
 ""serial_number"": ""123"",
 ""description"": ""Widget"",
 ""quantity"": 5,
 ""price"": 13.95
 }
]
}";
    }
}
```

`Main` 方法显示了如何使用自定义转换器、选项和策略：

```cs
static void Main()
{
    string poJson =
        new PurchaseOrderService()
            .Get(poID: 123);

    var jsonOptions = new JsonSerializerOptions
    {
        AllowTrailingCommas = true,
        Converters =
        {
            new PurchaseOrderStatusConverter()
        },
        PropertyNameCaseInsensitive = true,
        PropertyNamingPolicy = new SnakeCaseNamingPolicy(),
        WriteIndented = true
    };

    PurchaseOrder po =
        JsonSerializer
        .Deserialize<PurchaseOrder>(poJson, jsonOptions);

    Console.WriteLine($"{po.CompanyName}");
    Console.WriteLine($"{po.AdditionalInfo["terms"]}");
    Console.WriteLine($"{po.Items[0].Description}");

    string poJson2 = JsonSerializer.Serialize(po, jsonOptions);

    Console.WriteLine(poJson2);
}
```

这是输出结果：

```cs
Acme, Inc.
Net 30
Widget
{
  "company_name": "Acme, Inc.",
  "address": "123 4th St.",
  "phone": "555-835-7609",
  "status": "Processing",
  "additional_info": {
    "terms": "Net 30",
    "poc": "J. Smith"
  },
  "items": [
    {
      "serial_number": "123",
      "description": "Widget",
      "quantity": 5,
      "price": 13.95
    }
  ]
}
```

## 讨论

`JsonSerializer` 具有用于生成驼峰命名属性名的内置转换器，通过 `JsonInitializerOptions`，像这样：

```cs
var serializeOptions = new JsonSerializerOptions
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase
};
```

这处理了许多情况，但是如果第三方 API 没有使用帕斯卡命名或驼峰命名属性名称怎么办？此解决方案包括对由下划线分隔单词的蛇形命名属性名称的支持。例如，`SnakeCase` 变成了 `snake_case`。除了新的命名策略外，该解决方案还包括其他定制，包括对 `enum` 的支持。

注意，`PurchaseOrder` 不会使用 `JsonPropertyName` 装饰任何属性。相反，我们使用自定义命名策略，在 `SnakeCaseNamingPolicy` 类中定义，该类派生自 `JsonNamingPolicy`。`ConvertName` 中的算法假定它已收到帕斯卡命名规则的属性名。它迭代字符，查找大写字符。遇到大写字符时，它会附加下划线 `_`，将字母小写，并将小写字母附加到结果中。否则，它附加字符，该字符已经是小写的。

`Main` 方法实例化了 `JsonSerializerOptions`，将 `PropertyNamingPolicy` 设置为 `SnakeCaseNamingPolicy` 的实例。这将命名策略应用于所有属性名称，生成蛇形命名的属性名称。

###### 提示

与许多情况一样，您可能会遇到规则的例外情况，其中 JSON 属性不符合蛇形命名规则。在这种情况下，使用 `JsonPropertyName` 属性，如 Recipe 7.4 中所述，覆盖该属性的命名策略。

您可能已经注意到，在 `Main` 中，`JsonSerializerOptions` 还具有其他定制。`AllowTrailingCommas` 很有趣，因为有时您会收到包含列表的 JSON 数据，列表中的最后一项有一个尾随逗号。这会破坏反序列化，将 `AllowTrailingCommas` 设置为 `true` 可以忽略尾随逗号。

`PropertyNameCaseInsensitive` 是一种选择，不考虑属性名称的格式。在反序列化时，它允许小写属性名称与它们的大写等效项匹配。当传入的 JSON 属性名称可能不一致时，这是有用的。

默认情况下，`JsonSerializer` 生成单行 JSON 文档。设置 `WriteIndented` 可以格式化文本以提高可读性，如输出中所示。

其中一个属性 `Converters` 是一个类型集合，用于在属性上进行自定义转换。`PurchaseOrderStatusConverter` 从 `JsonConverter<T>` 派生，允许将 `Status` 属性反序列化为 `PurchaseOrderStatus` 枚举。有两种方法可以应用它：在 `JsonSerialization` 选项中或通过属性。将转换器添加到 `JsonSerializationOptions` 的 `Converter` 集合中会为所有 `PurchaseOrderStatus` 属性类型应用转换。此外，`PurchaseOrder` 类使用 `JsonConverter` 属性装饰 `Status` 属性。我在解决方案中添加了这两种方法，以便你能看到它们各自的工作方式。将转换器添加到 `Converters` 集合中就足够了。但是，如果你想要为特定属性应用不同的转换器，或者需要为不同的属性使用不同的转换器，那么请使用 `JsonConverter` 属性，因为它优先于 `Converters` 集合。

`Main` 方法展示了如何在反序列化和序列化中使用相同的 `JsonSerializationOptions`。

## 参见

7.4 节，“生成 JSON”

# 7.6 处理 JSON 数据

## 问题

收到了无法干净反序列化为对象的 JSON 数据。

## 解决方案

这是一个 `PurchaseOrder` 的样例：

```cs
public enum PurchaseOrderStatus
{
    Received,
    Processing,
    Fulfilled
}

public class PurchaseItem
{
    public string SerialNumber { get; set; }

    public string Description { get; set; }

    public double Quantity { get; set; }

    public decimal Price { get; set; }
}

public class PurchaseOrder
{
    public string CompanyName { get; set; }
    public string Address { get; set; }
    public string Phone { get; set; }
    public string Terms { get; set; }
    public string POC { get; set; }

    public PurchaseOrderStatus Status { get; set; }

    public Dictionary<string, string> AdditionalInfo { get; set; }

    public List<PurchaseItem> Items { get; set; }
}
```

这个类模拟了返回 JSON 数据的请求：

```cs
public class PurchaseOrderService
{
    public string Get(int poID)
    {
        // get HTTP request

        return @"{
""company_name"": ""Acme, Inc."",
""address"": ""123 4th St."",
""phone"": ""555-835-7609"",
""additional_info"": {
 ""terms"": ""Net 30"",
 ""poc"": ""J. Smith""
},
""status"": ""Processing"",
""items"": [
 {
 ""serial_number"": ""123"",
 ""description"": ""Widget"",
 ""quantity"": 5,
 ""price"": 13.95
 }
]
}";
    }
}
```

这里是支持自定义反序列化的类：

```cs
public static class JsonConversionExtensions
{
    public static bool IsNull(this JsonElement json)
    {
        return
            json.ValueKind == JsonValueKind.Undefined ||
            json.ValueKind == JsonValueKind.Null;
    }

    public static string GetString(
        this JsonElement json,
        string propertyName,
        string defaultValue = default)
    {
        if (!json.IsNull() &&
            json.TryGetProperty(propertyName, out JsonElement element))
            return element.GetString() ?? defaultValue;

        return defaultValue;
    }

    public static int GetInt(
        this JsonElement json,
        string propertyName,
        int defaultValue = default)
    {
        if (!json.IsNull() &&
            json.TryGetProperty(propertyName, out JsonElement element) &&
            !element.IsNull() &&
            element.TryGetInt32(out int value))
            return value;

        return defaultValue;
    }

    public static ulong GetULong(
        this string val,
        ulong defaultValue = default)
    {
        return string.IsNullOrWhiteSpace(val) ||
            !ulong.TryParse(val, out ulong result)
                ? defaultValue
                : result;
    }

    public static ulong GetUlong(
        this JsonElement json,
        string propertyName,
        ulong defaultValue = default)
    {
        if (!json.IsNull() &&
            json.TryGetProperty(propertyName, out JsonElement element) &&
            !element.IsNull() &&
            element.TryGetUInt64(out ulong value))
            return value;

        return defaultValue;
    }

    public static long GetLong(
        this JsonElement json,
        string propertyName,
        long defaultValue = default)
    {
        if (!json.IsNull() &&
            json.TryGetProperty(propertyName, out JsonElement element) &&
            !element.IsNull() &&
            element.TryGetInt64(out long value))
            return value;

        return defaultValue;
    }

    public static bool GetBool(
        this JsonElement json,
        string propertyName,
        bool defaultValue = default)
    {
        if (!json.IsNull() &&
            json.TryGetProperty(propertyName, out JsonElement element) &&
            !element.IsNull())
            return element.GetBoolean();

        return defaultValue;
    }

    public static double GetDouble(
        this string val,
        double defaultValue = default)
    {
        return string.IsNullOrWhiteSpace(val) ||
            !double.TryParse(val, out double result)
                ? defaultValue
                : result;
    }

    public static double GetDouble(
        this JsonElement json,
        string propertyName,
        double defaultValue = default)
    {
        if (!json.IsNull() &&
            json.TryGetProperty(propertyName, out JsonElement element) &&
            !element.IsNull() &&
            element.TryGetDouble(out double value))
            return value;

        return defaultValue;
    }

    public static decimal GetDecimal(
        this JsonElement json,
        string propertyName,
        decimal defaultValue = default)
    {
        if (!json.IsNull() &&
            json.TryGetProperty(propertyName, out JsonElement element) &&
            !element.IsNull() &&
            element.TryGetDecimal(out decimal value))
            return value;

        return defaultValue;
    }

    public static TEnum GetEnum<TEnum>
        (this JsonElement json,
        string propertyName,
        TEnum defaultValue = default)
        where TEnum: struct
    {
        if (!json.IsNull() &&
            json.TryGetProperty(propertyName, out JsonElement element) &&
            !element.IsNull())
        {
            string enumString = element.GetString();

            if (enumString != null &&
                Enum.TryParse(enumString, out TEnum num))
                return num;
        }

        return defaultValue;
    }
}
```

`Main` 方法展示了如何执行自定义反序列化：

```cs
static void Main()
{
    string poJson =
        new PurchaseOrderService()
            .Get(poID: 123);

    JsonElement elm = JsonDocument.Parse(poJson).RootElement;

    JsonElement additional = elm.GetProperty("additional_info");
    JsonElement items = elm.GetProperty("items");

    if (additional.IsNull() || items.IsNull())
        throw new ArgumentException("incomplete PO");

    var po = new PurchaseOrder
    {
        Address = elm.GetString("address", "none"),
        CompanyName = elm.GetString("company_name", string.Empty),
        Phone = elm.GetString("phone", string.Empty),
        Status = elm.GetEnum("status", PurchaseOrderStatus.Received),
        Terms = additional.GetString("terms", string.Empty),
        POC = additional.GetString("poc", string.Empty),
        AdditionalInfo =
            (from jElem in additional.EnumerateObject()
             select jElem)
            .ToDictionary(
                key => key.Name,
                val => val.Value.GetString()),
        Items =
            (from jElem in items.EnumerateArray()
             select new PurchaseItem
             {
                 Description = jElem.GetString("description"),
                 Price = jElem.GetDecimal("price"),
                 Quantity = jElem.GetDouble("quantity"),
                 SerialNumber = jElem.GetString("serial_number")
             })
            .ToList()
    };

    Console.WriteLine($"{po.CompanyName}");
    Console.WriteLine($"{po.Terms}");
    Console.WriteLine($"{po.AdditionalInfo["terms"]}");
    Console.WriteLine($"{po.Items[0].Description}");
}
```

## 讨论

虽然在序列化和反序列化中使用 `JsonSerializer` 是首选，但有时你不会得到 JSON 和 C# 对象之间干净的一对一结构匹配。例如，你可能需要从不同格式的不同源获取数据，并有一个单一的 C# 对象进行填充。其他时候，你可能有一个分层的 JSON 文档，并希望将其扁平化为你自己的对象。另一种常见情况是已经使用一个版本工作的对象，而 API 的新版本改变了结构。在某种程度上，这些都是同一个问题的多个视角，你可以通过自定义反序列化来解决。

自 `System.Text.Json` 命名空间中用于自定义反序列化的两种类型是 `JsonDocument` 和 `JsonElement`。`Main` 方法展示了如何使用 `JsonDocument` 解析 JSON 输入，并通过 `RootElement` 属性获取 `JsonElement`。之后，我们只需处理 `JsonElement` 的成员。

`JsonElement` 有多个成员，包括 `GetString` 和 `GetInt64`，用于进行转换。仅仅依赖这些成员存在的问题是数据通常不干净。即使你拥有应用程序的生产者和消费者端，获得完全干净的数据可能也是难以实现的。为了解决这个问题，我创建了 `JsonConversionExtensions` 类。

在概念上，`JsonConversionExtensions` 包装了许多模板代码，你需要调用它们来确保你正在读取的数据是你所期望的。它还有一个可选的默认值概念。

解决第一个问题的技巧是，`JsonElement`中的`null`值不表示为`null`。`IsNull`方法检查`ValueKind`属性，检查`Undefined`或`Null`属性是否为 true。这是其他转换方法中使用的重要方法。

浏览其余的方法，你会看到一个熟悉的模式。每个方法都会检查元素的`IsNull`，然后使用一个或多个`TryGetXxx`和`IsNull`的组合来获取值。这样做是安全的，在值为`null`或类型错误时避免异常。没错，一些 API 文档中的值是一个类型，运行时返回另一个类型，将数字设置为`null`，并省略属性。

每个方法都有一个默认参数。如果代码无法提取真实值，它将使用`defaultValue`。`defaultValue`参数是可选的，会回到返回类型的 C# `default`。

`Main`方法展示了如何使用`JsonElement`和`JsonConversionExtensions`类构造对象。你可以看到代码如何使用`GetXxx`方法填充每个属性。

几个有用的`JsonElement`方法是`EnumerateObject`和`EnumerateArray`。在前面的章节中，`JsonSerializer`将 JSON `additional_info`对象反序列化为 C#字典。这是处理具有可变信息对象的方法，你不知道对象属性是什么。在 API 返回单个错误 JSON 响应中，每个属性是一个错误的代码或描述。在`PurchaseOrder`示例中，这表示可以添加不适合预定义属性的杂项信息的地方。要手动读取这些属性，请使用`EnumerateObject`。它返回对象中的每个属性/值对。你可以看到 LINQ 语句如何通过从`EnumerateObject`返回的每个`JsonProperty`提取`Key`和`Value`来创建一个新字典。

`EnumerateArray`返回列表的每个元素。在解决方案中，我们将从`EnumerateArray`返回的每个`JsonElement`投影到一个新的`PurchaseOrderItem`实例中。

`JsonConversionExtensions`是不完整的，因为它不包括日期。由于`DateTime`处理是一个特例，我从这个示例中分离了它；你可以在 Recipe 7.10 中找到更多信息。

## 参见

Recipe 7.10，“灵活的 DateTime 读取”

# 7.7 消费 XML

## 问题

你需要将 XML 文档转换为对象。

## 解决方案

这是一个`PurchaseOrder`的样子：

```cs
public enum PurchaseOrderStatus
{
    Received,
    Processing,
    Fulfilled
}

public class PurchaseItem
{
    public string SerialNumber { get; set; }

    public string Description { get; set; }

    public float Quantity { get; set; }

    public decimal Price { get; set; }
}

public class PurchaseOrder
{
    public string CompanyName { get; set; }
    public string Address { get; set; }
    public string Phone { get; set; }

    public PurchaseOrderStatus Status { get; set; }

    public Dictionary<string, string> AdditionalInfo { get; set; }

    public List<PurchaseItem> Items { get; set; }
}
```

该方法模拟了返回 XML 数据的请求：

```cs
static string GetXml()
{
    return @"
<PurchaseOrder ">
 <Address>123 4th St.</Address>
 <CompanyName>Acme, Inc.</CompanyName>
 <Phone>555-835-7609</Phone>
 <Status>Received</Status>
 <AdditionalInfo>
 <Terms>Net 30</Terms>
 <POC>J. Smith</POC>
 </AdditionalInfo>
 <Items>
 <PurchaseItem SerialNumber=""123"">
 <Description>Widget</Description>
 <Price>13.95</Price>
 <Quantity>5</Quantity>
 </PurchaseItem>
 </Items>
</PurchaseOrder>";
}
```

`Main`方法展示了如何将 XML 反序列化为对象：

```cs
static void Main(string[] args)
{
    XNamespace or = "https://www.oreilly.com";

    XName address = or + nameof(PurchaseOrder.Address);
    XName company = or + nameof(PurchaseOrder.CompanyName);
    XName phone = or + nameof(PurchaseOrder.Phone);
    XName status = or + nameof(PurchaseOrder.Status);
    XName info = or + nameof(PurchaseOrder.AdditionalInfo);
    XName poItems = or + nameof(PurchaseOrder.Items);
    XName purchaseItem = or + nameof(PurchaseItem);
    XName description = or + nameof(PurchaseItem.Description);
    XName price = or + nameof(PurchaseItem.Price);
    XName quantity = or + nameof(PurchaseItem.Quantity);
    XName serialNum = nameof(PurchaseItem.SerialNumber);

    string poXml = GetXml();

    XElement poElmt = XElement.Parse(poXml);

    PurchaseOrder po =
        new PurchaseOrder
        {
            Address = (string)poElmt.Element(address),
            CompanyName = (string)poElmt.Element(company),
            Phone = (string)poElmt.Element(phone),
            Status =
                Enum.TryParse(
                    (string)poElmt.Element(nameof(po.Status)),
                    out PurchaseOrderStatus poStatus)
                ? poStatus
                : PurchaseOrderStatus.Received,
            AdditionalInfo =
                (from addInfo in poElmt.Element(info).Descendants()
                 select addInfo)
                .ToDictionary(
                    key => key.Name.LocalName,
                    val => val.Value),
            Items =
                (from item in poElmt
                                .Element(poItems)
                                .Descendants(purchaseItem)
                 select new PurchaseItem
                 {
                     Description = (string)item.Element(description),
                     Price =
                        decimal.TryParse(
                            (string)item.Element(price),
                            out decimal itemPrice)
                        ? itemPrice
                        : 0m,
                     Quantity =
                        float.TryParse(
                            (string)item.Element(quantity),
                            out float qty)
                        ? qty
                        : 0f,
                     SerialNumber = (string)item.Attribute(serialNum)
                 })
                .ToList()
        };

    Console.WriteLine($"{po.CompanyName}");
    Console.WriteLine($"{po.AdditionalInfo["Terms"]}");
    Console.WriteLine($"{po.Items[0].Description}");
    Console.WriteLine($"{po.Items[0].SerialNumber}");
}
```

## 讨论

在 JSON 成为主导数据格式之前，XML 无处不在。在处理配置文件、项目文件或可扩展应用标记语言（XAML）等方面，XML 仍然非常明显。还有相当数量的遗留代码，包括广泛使用 XML 的 Windows Communication Foundation（WCF）Web 服务。暂时而言，掌握如何处理 XML 是一项宝贵的技能，而 LINQ to XML 是一个优秀的工具。

解决方案展示了如何将 XML 反序列化为 `PurchaseOrder` 对象。`Main` 方法首先设置命名空间。XML 中的命名空间很常见，代码创建了一个命名空间标签 `or`。`XNamespace` 类型有一个转换器，将字符串转换为命名空间。`XNamespace` 还重载了 `+` 运算符，允许您给元素打上特定的命名空间标签，创建一个新的 `XName`。代码为每个元素设置了一个 `XName`，以使 `PurchaseOrder` 的构造更易于阅读。

每个元素都有一个命名空间，`serialNum` 除外，它是一个属性。数据属性不需要用命名空间注释，因为它们位于包含元素的命名空间中。唯一的例外是，如果您想要向元素添加命名空间属性，将其放入新的命名空间中。

在获取 XML 后，`Main` 调用 `XElement.Parse` 获取一个新的 `XElement` 来处理。`XElement` 具有移动文档和读取所需内容所需的所有轴方法。此示例通过使用 `Attribute`、`Element` 和 `Descendants` 轴方法按层次移动文档来保持简单。

`Element` 方法帮助读取当前元素下的子元素。`Descendants` 方法深入一级，访问指定元素的子元素。从 `GetXml` 返回的 XML 中，`PurchaseOrder` 是根元素，由 `poElmt` 表示。查看 `PurchaseOrder`，`poElmt.Element(address)` 读取 `PurchaseOrder` 的子元素 `Address`。如您所知，`address` 是一个命名空间限定的 `XName`。

填充 `AdditionalInfo` 和 `Items` 属性展示了如何使用 `Descendants`。我们使用 `Element` 来读取子元素，使用 `Descendants` 来获取该元素的子元素列表。对于 `AdditionalInfo`，`Descendants` 是可变元素和值，并且我们不传递 `XName` 参数。对于 `Items`，我们需要传递 `purchaseItem` 的 `XName` 给 `Descendants`，以便对每个对象进行操作。

我们使用 `Attribute` 方法来填充每个 `PurchaseOrderItem` 的 `SerialNumber` 属性。

此对象构造的有趣部分是在 `TryParse` 操作中声明 `out` 参数的能力。这使我们可以在对象构造时内联编码分配。在此 C# 特性之前，我们需要在对象构造外部声明变量，这在像解决方案中的 LINQ 投影中填充 `Items` 属性时并不自然。

## 参见

Recipe 7.8, “生成 XML”

# 7.8 XML 生成

## 问题

你需要将一个对象转换为 XML。

## 解决方案

这是一个`PurchaseOrder`的样子：

```cs
public enum PurchaseOrderStatus
{
    Received,
    Processing,
    Fulfilled
}

public class PurchaseItem
{
    public string SerialNumber { get; set; }

    public string Description { get; set; }

    public float Quantity { get; set; }

    public decimal Price { get; set; }
}

public class PurchaseOrder
{
    public string CompanyName { get; set; }
    public string Address { get; set; }
    public string Phone { get; set; }

    public PurchaseOrderStatus Status { get; set; }

    public Dictionary<string, string> AdditionalInfo { get; set; }

    public List<PurchaseItem> Items { get; set; }
}
```

这个方法模拟一个数据请求，返回一个`PurchaseOrder`：

```cs
static PurchaseOrder GetPurchaseOrder()
{
    return new PurchaseOrder
    {
        CompanyName = "Acme, Inc.",
        Address = "123 4th St.",
        Phone = "555-835-7609",
        AdditionalInfo = new Dictionary<string, string>
        {
            { "Terms", "Net 30" },
            { "POC", "J. Smith" }
        },
        Items = new List<PurchaseItem>
        {
            new PurchaseItem
            {
                Description = "Widget",
                Price = 13.95m,
                Quantity = 5,
                SerialNumber = "123"
            }
        }
    };
}
```

`Main`方法展示了如何将`PurchaseOrder`实例序列化为 XML：

```cs
static void Main(string[] args)
{
    PurchaseOrder po = GetPurchaseOrder();

    XNamespace or = "https://www.oreilly.com";

    XElement poXml =
        new XElement(or + nameof(PurchaseOrder),
            new XElement(
                or + nameof(PurchaseOrder.Address),
                po.Address),
            new XElement(
                or + nameof(PurchaseOrder.CompanyName),
                po.CompanyName),
            new XElement(
                or + nameof(PurchaseOrder.Phone),
                po.Phone),
            new XElement(
                or + nameof(PurchaseOrder.Status),
                po.Status),
            new XElement(
                or + nameof(PurchaseOrder.AdditionalInfo),
                (from info in po.AdditionalInfo
                 select
                     new XElement(
                         or + info.Key,
                         info.Value))
                .ToList()),
            new XElement(
                or + nameof(PurchaseOrder.Items),
                (from item in po.Items
                 select new XElement(
                     or + nameof(PurchaseItem),
                     new XAttribute(
                         nameof(PurchaseItem.SerialNumber),
                         item.SerialNumber),
                     new XElement(
                         or + nameof(PurchaseItem.Description),
                         item.Description),
                     new XElement(
                         or + nameof(PurchaseItem.Price),
                         item.Price),
                     new XElement(
                         or + nameof(PurchaseItem.Quantity),
                         item.Quantity)))
                .ToList()));

    Console.WriteLine(poXml);
}
```

这里是输出结果：

```cs
<PurchaseOrder xmlns="https://www.oreilly.com">
  <Address>123 4th St.</Address>
  <CompanyName>Acme, Inc.</CompanyName>
  <Phone>555-835-7609</Phone>
  <Status>Received</Status>
  <AdditionalInfo>
    <Terms>Net 30</Terms>
    <POC>J. Smith</POC>
  </AdditionalInfo>
  <Items>
    <PurchaseItem SerialNumber="123">
      <Description>Widget</Description>
      <Price>13.95</Price>
      <Quantity>5</Quantity>
    </PurchaseItem>
  </Items>
</PurchaseOrder>
```

## 讨论

Recipe 7.7 将一个 XML 文档反序列化为一个`PurchaseOrder`对象。这一部分则相反—将`PurchaseOrder`序列化为 XML 文档。

我们从`XNamespace`开始，用作每个元素的`XName`参数，以保持所有元素在同一个命名空间中。

解决方案通过调用`XElement`和`XAttribute`构建 XML 文档。我们唯一使用`XAttribute`的地方是每个`PurchaseOrderItem`元素的`SerialNumber`属性。

从视觉上看，你可以看到 LINQ 到 XML 查询子句的布局与其生成的 XML 输出具有相同的分层结构。解决方案使用了两个`XElement`构造函数重载。如果一个元素是一个底层节点，没有子元素，第二个参数是元素的值。然而，如果元素是一个父元素，有子元素，第二个参数是一个新的`XElement`。

LINQ 语句用于`AdditionalInfo`和`Items`，生成一个新的`XElement`。

## 参见

Recipe 7.7, “消费 XML”

# 7.9 编码和解码 URL 参数

## 问题

你正在使用一个需要符合 RFC 3986 的 API 进行操作。

## 解决方案

这里是一个正确编码 URL 参数的类：

```cs
public class Url
{
    /// <summary>
    /// Implements Percent Encoding according to RFC 3986
    /// </summary>
    /// <param name="value">string to be encoded</param>
    /// <returns>Encoded string</returns>
    public static string PercentEncode(
        string? value, bool isParam = true)
    {
        const string IsParamReservedChars = @"`!@#$^&*+=,:;'?/|\[] ";
        const string NoParamReservedChars = @"`!@#$^&*()+=,:;'?/|\[] ";

        var result = new StringBuilder();

        if (string.IsNullOrWhiteSpace(value))
            return string.Empty;

        var escapedValue = EncodeDataString(value);

        var reservedChars =
            isParam ? IsParamReservedChars : NoParamReservedChars;

        foreach (char symbol in escapedValue)
        {
            if (reservedChars.IndexOf(symbol) != -1)
                result.Append(
                    '%' +
                    string.Format("{0:X2}", (int)symbol).ToUpper());
            else
                result.Append(symbol);
        }

        return result.ToString();
    }

    /// <summary>
    /// URL-encode a string of any length.
    /// </summary>
    static string EncodeDataString(string data)
    {
        // the max length in .NET 4.5+ is 65520
        const int maxLength = 65519;

        if (data.Length <= maxLength)
            return Uri.EscapeDataString(data);

        var totalChunks = data.Length / maxLength;

        var builder = new StringBuilder();
        for (var i = 0; i <= totalChunks; i++)
        {
            string? chunk =
                i < totalChunks ?
                    data[(maxLength * i)..maxLength] :
                    data[(maxLength * i)..];

            builder.Append(Uri.EscapeDataString(chunk));
        }
        return builder.ToString();
    }
}
```

此方法解析 URL，编码参数并重建 URL：

```cs
static string EscapeUrlParams(string originalUrl)
{
    const int Base = 0;
    const int Parms = 1;
    const int Key = 0;
    const int Val = 1;
    string[] parts = originalUrl.Split('?');
    string[] pairs = parts[Parms].Split('&');

    string escapedParms =
        string.Join('&',
            (from pair in pairs
             let keyVal = pair.Split('=')
             let encodedVal = Url.PercentEncode(keyVal[Val])
             select $"{keyVal[Key]}={encodedVal}")
            .ToList());

    return $"{parts[Base]}?{escapedParms}";
}
```

`Main`方法比较不同的编码方式：

```cs
static void Main()
{
    const string OriginalUrl =
        "https://myco.com/po/search?company=computers+";
    Console.WriteLine($"Original:    '{OriginalUrl}'");

    string escapedUri = Uri.EscapeUriString(OriginalUrl);
    Console.WriteLine($"Escape URI:  '{escapedUri}'");

    string escapedData = Uri.EscapeDataString(OriginalUrl);
    Console.WriteLine($"Escape Data: '{escapedData}'");

    string escapedUrl = EscapeUrlParams(OriginalUrl);
    Console.WriteLine($"Escaped URL: '{escapedUrl}'");
}
```

生成这个输出：

```cs
Original:    'https://myco.com/po/search?company=computers+'
Escape URI:  'https://myco.com/po/search?company=computers+'
Escape Data: 'https%3A%2F%2Fmyco.com%2Fpo%2Fsearch%3Fcompany
%3Dcomputers%2B'
Escaped URL: 'https://myco.com/po/search?company=computers%2B'
```

## 讨论

如果你同时构建网络通信的消费者和生产者部分，比如内部企业应用，编码的正确性可能并不重要，因为这两部分使用同一个库。然而，某些第三方 API 要求严格遵守 RFC 3986。你可能首先想到的是.NET 中的`System.Uri`类有`EscapeUriString`和`EscapeDataString`方法。不幸的是，这些方法并没有始终正确实现 RFC 3986。虽然.NET 5+跨平台并且看起来实现良好，但是.NET Framework 的早期版本针对不同技术并没有这样做。为了解决这个问题，我在解决方案中创建了`Url`类。

###### 注意

RFC 3986 是定义互联网 URL 编码的标准。RFC 代表“请求评论”，标准通常以 RFC 后跟一些唯一编号。

`PercentEncode` 将值参数的每个字符替换为带有百分号（`%`）前缀的两位十六进制表示。第一个操作是调用 `EscapeDataString`。该方法调用 `Uri.EscapeDataString`。`Uri.EscapeDataString` 的一个问题是长度限制，因此该方法会将输入分块以确保所有数据都被编码。方法的思路是让 `Uri.EscapeDataString` 处理大部分的转换工作，并让 `PercentEncode` 补充那些未被编码的字符。

`PercentEncode` 还有一个第二个参数 `isParam`，用于指示是否应编码括号。它默认为 `true`，用户可以将其设置为 `false` 以防止编码括号，这是 `IsParamReservedChars` 和 `NoParamReservedChars` 之间唯一的区别。如果该方法发现未编码的字符，它会手动进行编码。

我们只对查询字符串参数值进行编码，因为基本 URL、段和参数名称是不需要编码的值。`EscapeUrlParameters` 方法通过将 URL 与参数分离，并迭代每个参数来实现此目的。对于每个参数，它将参数名与其值分开，并对值调用 `PercentEncode`。在对值进行编码之后，代码重建并返回 URL。

`Main` 方法展示了不同类型的编码，阐明了为什么选择了自定义编码方法。请注意，`Uri.EscapeUriString` 没有对 `+` 符号进行编码。使用 `Uri.EscapeDataString` 对整个 URL 进行了编码，这并不是您想要的。将 URL 拆分并对每个值进行编码可以完美解决问题。

请记住，在 .NET 5+ 应用程序中可能会获得良好的结果。但是，如果在旧的 .NET Framework 版本中进行跨平台工作，`Uri.EscapeUriString` 和 `Uri.EscapeDataString` 的结果可能不一致，很可能会导致错误。无论使用哪个框架/技术版本，仅对参数值进行编码的技术是一个常见的需求。

# 7.10 灵活的日期时间读取

## 问题

您需要解析可能以多种不同格式出现的 `DateTime` 值。

## 解决方案

这些扩展方法有助于解析日期：

```cs
public static class StringExtensions
{
    static readonly string[] dateFormats =
    {
        "ddd MMM dd HH:mm:ss %zzzz yyyy",
        "yyyy-MM-dd\\THH:mm:ss.000Z",
        "yyyy-MM-dd\\THH:mm:ss\\Z",
        "yyyy-MM-dd HH:mm:ss",
        "yyyy-MM-dd HH:mm"
    };

    public static DateTime GetDate(
        this string date,
        DateTime defaultValue)
    {
        return string.IsNullOrWhiteSpace(date) ||
            !DateTime.TryParseExact(
                    date,
                    dateFormats,
                    CultureInfo.InvariantCulture,
                    DateTimeStyles.AssumeUniversal |
                    DateTimeStyles.AdjustToUniversal,
                    out DateTime result)
                ? defaultValue
                : result;
    }

    public static DateTime GetDate(
        this JsonElement json,
        string propertyName,
        DateTime defaultValue = default)
    {
        string? date = json.GetString(propertyName);
        return date?.GetDate(defaultValue) ?? defaultValue;
    }

    public static string? GetString(
        this JsonElement json,
        string propertyName,
        string? defaultValue = default)
    {
        if (!json.IsNull() &&
            json.TryGetProperty(propertyName, out JsonElement element))
            return element.GetString() ?? defaultValue;

        return defaultValue;
    }

    public static bool IsNull(this JsonElement json)
    {
        return
            json.ValueKind == JsonValueKind.Undefined ||
            json.ValueKind == JsonValueKind.Null;
    }
}
```

`Main` 方法展示了如何提取和解析 JSON 文档中的日期：

```cs
static void Main()
{
    const string TweetID = "1305895383260782593";
    const string CreatedDate = "created_at";

    string tweetJson = GetTweet(TweetID);

    JsonElement tweetElem = JsonDocument.Parse(tweetJson).RootElement;

    DateTime created = tweetElem.GetDate(CreatedDate);

    Console.WriteLine($"Created Date: {created}");
}

static string GetTweet(string tweetID)
{
    return @"{
 ""text"": ""Thanks @github for approving sponsorship for
 LINQ to Twitter: https://t.co/jWeDEN07HN"",
 ""id"": ""1305895383260782593"",
 ""author_id"": ""15411837"",
 ""created_at"": ""2020-09-15T15:44:56.000Z""
 }";
}
```

## 讨论

在使用第三方 API 时，您可能会遇到数据表示的偶发不一致。一个问题区域是解析日期。不同的 API 使用不同的日期格式，甚至在同一个 API 中，也可能用不同的格式表示不同的日期属性。解决方案中的 `StringExtensions` 类帮助解决了这个问题。

###### 注意

我从 7.6 节 中的 `JsonConversionExtensions` 中提取了 `StringExtensions` 成员。

解决方案包括一个`dateFormats`数组，其中包含日期格式字符串的实例。这些都是此代码可以容纳的所有可能日期格式。`GetDate`方法在调用`TryParseExact`时使用`dateFormats`。每当遇到新的日期格式（例如，如果 API 提供了新版本并更新了日期格式），请将其添加到`dateFormats`中。

最佳实践是将日期表示为 UTC 值，因此`DateTimeStyles`参数反映了这一假设。

函数`GetDate`有两个重载，取决于您需要传递`string`还是`JsonElement`。`JsonElement`的重载使用`GetString`扩展方法，并将结果转发给另一个`GetDate`方法。

这些方法是安全的，因为您必须考虑糟糕的数据。它们检查`null`，使用`TryParse`，并在无法读取有效值时返回`default`值。如果未提供`defaultValue`，则返回类型的`default`是可选的。

## 参见

Recipe 7.6，“使用 JSON 数据”
