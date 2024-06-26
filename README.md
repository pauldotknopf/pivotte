# Overview

```Pivotte``` is a library that helps you declare your APIs separately from your implementation. Think of it like Swagger, but in C#, with type safety.

```csharp
public class GetProductRequest
{
    public string Country { get; set; }
}

[PivotteService("Product")]
public interface IProductService
{
    [Route("get/{id}")]
    [HttpPost]
    Product GetProduct([FromRoute]int id, [FromBody]GetProductRequest request);
}
```

Auto wire-up the definition to an implementation, using ASP.NET Minimal APIs.

```csharp
public class ProductService : IProductService
{
    public Product GetProduct(int id, GetProductRequest request)
    {
        return new Product
        {
            Id = id,
            Name = "Product " + id + " " + request.Country,
            Price = id * 10f
        };
    }
}
// ...
builder.Services.AddSingleton<IProductService, ProductService>();
// ...
app.MapPivotteService<IProductService>("api"); // endpoint "/api/get/{id}
// ...
```

With ```Pivotte.Generators```, you can generate client-side code (supported by NSwag).

```csharp
var generator = sp.GetService<IGenerator>();
var code = generator.GenerateClientTypeScript(config =>
{
    config.AddService<IProductService>();
});
// save 'code' wherever makes sense to you
```

With ```Pivotte.NetClient```, you can invoke an API from another .NET project with the same api definition as well.

```csharp
var clientGenerator = sp.GetService<IPivotteClientGenerator>();
var httpClient = new HttpClient
{
    BaseAddress = new Uri("https://somewhere.com/api")
};
// "client" is a runtime-generated implementation of IProductService.
// invoking methods will cause an HTTP request underneath the hood.
var client = clientGenerator.Generate<IProductService>(httpClient);
var product = await client.GetProduct(3, new GetProductRequest
{
    Country = "US"
});
```

# Why?

Maybe you are a large organization with many APIs. Having the definition of each API in a single repository can allow teams to easily communicate their changes (using PRs). Having them in the same repository would also allow shared types across different APIs, ensuring consistency/uniformity.

```csharp
[PivotteService("MicroServiceOne")]
public interface IMicroServiceOne
{
    [Route("dostuff")]
    [HttpPost]
    void DoStuff();
}
[PivotteService("MicroServiceTwo")]
public interface IMicroServiceTwo
{
    [Route("dostuff")]
    [HttpPost]
    void DoStuff();
}
```

This single repository could also auto-deploy an NPM package containing the TypeScript definitions for invoking the APIs.
