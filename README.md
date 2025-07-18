﻿# Iciclecreek.SemanticKernel.Connectors.Files
This library defines a file based Vector Store provider for Microsoft.Extensions.VectorData

## Details
This library defines FileVectoreStore which is a Vector Store provider that uses files to store the vectors and metadata.

> FileVectorStore is a wrapper around InMemoryVectorStore, but with file persistance. It is useful for unit tests and local development, but probably not appropriate for large production work loads.

## 🚀 Getting Started with FileVectorStore

### 1. Install NuGet Packages

```bash
dotnet add package Microsoft.Extensions.VectorData.Abstractions
dotnet add package Iciclecreek.SemanticKernel.Connectors.Files
```

---

### 2. Define Your Data Model

```csharp
using Microsoft.Extensions.VectorData;

public class Hotel
{
    [VectorStoreKey]
    public ulong HotelId { get; set; }

    [VectorStoreData(IsIndexed = true)]
    public string HotelName { get; set; }

    [VectorStoreData(IsFullTextIndexed = true)]
    public string Description { get; set; }

    [VectorStoreVector(Dimensions: 4, DistanceFunction = DistanceFunction.CosineSimilarity, IndexKind = IndexKind.Hnsw)]
    public ReadOnlyMemory<float>? DescriptionEmbedding { get; set; }

    [VectorStoreData(IsIndexed = true)]
    public string[] Tags { get; set; }
}
```

---

### 3. Instantiate a FileVectorStore

```csharp
using Microsoft.SemanticKernel.Connectors.Files;

// Create a FileVectorStore instance pointing to a local directory
var vectorStore = new FileVectorStore(new FileVectorStoreOptions() 
    { 
        Path = @"C:\\path\\to\\your\\vectorstore" // Specify your directory path here,
        EmbeddingGenerator =...
    });

// Get a collection for Hotel records
var collection = vectorStore.GetCollection<ulong, Hotel>("skhotels");
```

---

### 4. Create Collection and Add Records

```csharp
// Ensure collection exists
await collection.EnsureCollectionExistsAsync();

// Add a record
ulong hotelId = 1;
string description = "A place where everyone can be happy.";

await collection.UpsertAsync(new Hotel
{
    HotelId = hotelId,
    HotelName = "Hotel Happy",
    Description = description,
    Tags = new[] { "luxury", "pool" }
});

// Retrieve the record
Hotel? retrieved = await collection.GetAsync(hotelId);
```

---

### 5. Perform a Vector Search

```csharp

await foreach (var result in collection.SearchAsync("I'm looking for a hotel where customer happiness is the priority."))
{
    Console.WriteLine($"Found hotel: {result.Record.Description}");
    Console.WriteLine($"Score: {result.Score}");
}
```

## Dependency Injection extensions

This library provides extension methods to register the `FileVectorStore` with Microsoft.Extensions.DependencyInjection.

```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.SemanticKernel.Connectors.Files;

// Register the FileVectorStore with DI
services.AddFileVectorStore(options => 
{
    options.Path = @"C:\\path\\to\\your\\vectorstore"; // Specify your directory path here
    options.EmbeddingGenerator = ...; // Provide your embedding generator here
});
```
