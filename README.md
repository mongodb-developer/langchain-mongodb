Setup and Installation
Install the Node MongoDB SDK in your project:

sh
Copy code
npm install -S mongodb
Create a MongoDB Atlas cluster by signing up on the MongoDB Atlas website. Create and name a cluster, and select or create a collection within the cluster.

Create an index on the collection field you want to search over. This index should specify the dimensions of the embeddings you are using, similarity type, and field names.

json
Copy code
{
  "mappings": {
    "fields": {
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
Usage: Ingestion
javascript
Copy code
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
    indexName: "default",
    textKey: "text",
    embeddingKey: "embedding"
  }
);

await client.close();
Usage: Similarity Search
javascript
Copy code
const vectorStore = new MongoDBAtlasVectorSearch(new CohereEmbeddings(), {
  collection,
  indexName: "default",
  textKey: "text",
  embeddingKey: "embedding"
});

const resultOne = await vectorStore.similaritySearch("Hello world", 1);
console.log(resultOne);

await client.close();
Usage: Maximal Marginal Relevance (MMR) Search
javascript
Copy code
const vectorStore = new MongoDBAtlasVectorSearch(new CohereEmbeddings(), {
  collection,
  indexName: "default",
  textKey: "text",
  embeddingKey: "embedding"
});

const resultOne = await vectorStore.maxMarginalRelevanceSearch("Hello world", {
  k: 4,
  fetchK: 20
});
console.log(resultOne);

// Using MMR in a vector store retriever
const retriever = await vectorStore.asRetriever({
  searchType: "mmr",
  searchKwargs: {
    fetchK: 20,
    lambda: 0.1,
  },
});

const retrieverOutput = await retriever.getRelevantDocuments("Hello world");

console.log(retrieverOutput);

await client.close();
These code snippets demonstrate how to set up LangChain.js with MongoDB Atlas as the vector store and perform similarity and MMR searches. You need to replace specific values, such as the MongoDB Atlas URI, with your own configuration and data. Additionally, ensure that you have the required packages and modules installed in your project.
