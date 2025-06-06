# Up-to-date Examples
For up-to-date examples, check out [MongoDB's Gen AI Showcase](https://github.com/mongodb-developer/GenAI-Showcase/).

# Notice: Repository Deprecation
This repository is deprecated and no longer actively maintained. It contains outdated code examples or practices that do not align with current MongoDB best practices. While the repository remains accessible for reference purposes, we strongly discourage its use in production environments.
Users should be aware that this repository will not receive any further updates, bug fixes, or security patches. This code may expose you to security vulnerabilities, compatibility issues with current MongoDB versions, and potential performance problems. Any implementation based on this repository is at the user's own risk.
For up-to-date resources, please refer to the [MongoDB Developer Center](https://mongodb.com/developer).

<img src="https://webimages.mongodb.com/_com_assets/cms/lmasvfr5imbft2sbf-LangChain_Workmark_Color%201.svg?auto=format%252Ccompress" alt="LangChain Logo" width="50%" />

Setup and Installation (this is all nodejs so grab that if you don't have it installed)

First, add the Node MongoDB SDK to your project using one of the package managers:

```
npm install -S mongodb
```

Initial Cluster Configuration
Create a MongoDB Atlas cluster by signing up on the MongoDB Atlas website if you haven't already.

Create and name a cluster when prompted and find it under the Database section. Select Collections and create either a blank collection or use provided sample data.

Creating an Index
After configuring your cluster, create an index on the collection field you want to search over. Use the JSON editor option and add an index to the desired collection:

```
{
  "mappings": {
    "fields": {
      // Default value, should match the name of the field within your collection that contains embeddings
      "embedding": [
        {
          "dimensions": 1024,
          "similarity": "euclidean",
          "type": "knnVector"
        }
      ]
    }
  }
}
```

The dimensions property should match the dimensionality of the embeddings you are using (e.g., Cohere embeddings have 1024 dimensions, and OpenAI embeddings have 1536).

Note: By default, the vector store expects an index name of default, an indexed collection field name of embedding, and a raw text field name of text. Initialize the vector store with field names matching your collection schema as shown below.

Finally, proceed to build the index.

```
import { MongoDBAtlasVectorSearch } from "langchain/vectorstores/mongodb_atlas";
import { CohereEmbeddings } from "langchain/embeddings/cohere";
import { MongoClient } from "mongodb";

const client = new MongoClient(process.env.MONGODB_ATLAS_URI || "");
const namespace = "langchain.test";
const [dbName, collectionName] = namespace.split(".");
const collection = client.db(dbName).collection(collectionName);

await MongoDBAtlasVectorSearch.fromTexts(
  ["Hello world", "Bye bye", "What's this?"],
  [{ id: 2 }, { id: 1 }, { id: 3 }],
  new CohereEmbeddings(),
  {
    collection,
    indexName: "default", // The name of the Atlas search index. Defaults to "default"
    textKey: "text", // The name of the collection field containing the raw content. Defaults to "text"
    embeddingKey: "embedding", // The name of the collection field containing the embedded text. Defaults to "embedding"
  }
);

await client.close();
```

Usage: Similarity Search https://www.earthdata.nasa.gov/dashboard/labs/similarity-search/

```
import { MongoDBAtlasVectorSearch } from "langchain/vectorstores/mongodb_atlas";
import { CohereEmbeddings } from "langchain/embeddings/cohere";
import { MongoClient } from "mongodb";

const client = new MongoClient(process.env.MONGODB_ATLAS_URI || "");
const namespace = "langchain.test";
const [dbName, collectionName] = namespace.split(".");
const collection = client.db(dbName).collection(collectionName);

const vectorStore = new MongoDBAtlasVectorSearch(new CohereEmbeddings(), {
  collection,
  indexName: "default", // The name of the Atlas search index. Defaults to "default"
  textKey: "text", // The name of the collection field containing the raw content. Defaults to "text"
  embeddingKey: "embedding", // The name of the collection field containing the embedded text. Defaults to "embedding"
});

const resultOne = await vectorStore.similaritySearch("Hello world", 1);
console.log(resultOne);

await client.close();
```

Usage: Maximal Marginal Relevance (MMR) Search (Not MaximumRocknRoll) ☠️

```
import { MongoDBAtlasVectorSearch } from "langchain/vectorstores/mongodb_atlas";
import { CohereEmbeddings } from "langchain/embeddings/cohere";
import { MongoClient } from "mongodb";

const client = new MongoClient(process.env.MONGODB_ATLAS_URI || "");
const namespace = "langchain.test";
const [dbName, collectionName] = namespace.split(".");
const collection = client.db(dbName).collection(collectionName);

const vectorStore = new MongoDBAtlasVectorSearch(new CohereEmbeddings(), {
  collection,
  indexName: "default", // The name of the Atlas search index. Defaults to "default"
  textKey: "text", // The name of the collection field containing the raw content. Defaults to "text"
  embeddingKey: "embedding", // The name of the collection field containing the embedded text. Defaults to "embedding"
});

```
```
const resultOne = await vectorStore.maxMarginalRelevanceSearch("Hello world", {
  k: 4,
  fetchK: 20, // The number of documents to return on the initial fetch
});
console.log(resultOne);
```

// Using MMR in a vector store retriever

```
const retriever = await vectorStore.asRetriever({
  searchType: "mmr",
  searchKwargs: {
    fetchK: 20,
    lambda: 0.1,
  },
});
```
```
const retrieverOutput = await retriever.getRelevantDocuments("Hello world");
console.log(retrieverOutput);
await client.close();
```

These code snippets provide instructions and examples for setting up LangChain.js with MongoDB Atlas as a vector store for similarity and maximal marginal relevance (MMR) search.


[Langchain Embeddings](https://python.langchain.com/docs/modules/data_connection/text_embedding/)  🦜

[MongoDB Vector Search](https://js.langchain.com/docs/api/vectorstores_mongodb_atlas/classes/MongoDBAtlasVectorSearch)  🍃

