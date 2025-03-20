# Toy Store Search App

## Set up AlloyDB Database
AlloyDB is more than 4x faster for transactional workloads and provides up to 2x better price-performance compared to self-managed PostgreSQL.

It uses the Google ScaNN index, the technology that powers services like Google Search and YouTube, to scale to over a billion vectors and deliver up to 4 times faster vector queries than the HNSW index in standard PostgreSQL.

It also helps you generate vector embeddings from within your database.

AlloyDB Omni is a downloadable edition of AlloyDB, designed to run anywhere—in your data center, on your laptop, at the edge, and in any cloud. It’s powered by the same engine that underlies the cloud-based AlloyDB service and provides the same functionality. AlloyDB Omni is a fraction of the cost of legacy databases, so it’s an attractive way to modernize to an enterprise-grade version of PostgreSQL with support from a Tier 1 vendor.


### AlloyDB
100% PostgreSQL-compatible database that runs anywherev

Follow steps in step 4 in this codelab: https://codelabs.developers.google.com/smart-shop-agent-alloydb#3
Remember to name the cluster and instance as follows:
cluster: vector-cluster
instances: vector-instance
change the user, password, db_name in your Cloud Run Function application code (in the "get-toys-alloydb" Cloud Run Function) according to what you set in this setup step.

### 1. CREATE Script

#### CREATE TABLE toys ( id VARCHAR(25), name VARCHAR(25), description VARCHAR(20000), quantity INT, price FLOAT, image_url VARCHAR(200), text_embeddings vector(768)) ;


### 2. INSERT Script
INSERT SCRIPTS in the file data.sql in this repo

### 3. Enable Extensions

#### CREATE EXTENSION vector;

#### CREATE EXTENSION google_ml_integration;


### 4. Grant Permission

#### GRANT EXECUTE ON FUNCTION embedding TO postgres;


### 5. Grant Vertex AI User ROLE to the AlloyDB service account

#### PROJECT_ID=$(gcloud config get-value project)

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:service-$(gcloud projects describe $PROJECT_ID --format="value(projectNumber)")@gcp-sa-alloydb.iam.gserviceaccount.com" \
--role="roles/aiplatform.user"

### 6. Update Text Embeddings

#### UPDATE toys set text_embeddings = embedding( 'text-embedding-005', description);

### 7. Vector Search (RAG)

At this point you are ready to test your Nearest Neighbor query results using the embeddings just created from AlloyDB Studio.
For detailed steps on how to create ScaNN index, refer to this blog: 

####    select * from toys
####    ORDER BY text_embeddings <=> CAST(embedding('text-embedding-005', 'white plush teddy bear toy with floral pattern') as vector(768))
####    LIMIT 5;

### 7. ScaNN Index

[ScaNN]([url](https://github.com/google-research/google-research/blob/master/scann%2FREADME.md)), a state-of-the-art approximate nearest neighbor search algorithm, offers a powerful solution for optimizing vector search performance. It excels at handling large-scale datasets and complex queries, making it particularly well-suited for retail use cases. ScaNN employs a three-phase approach to vector search:

#### Partitioning: 
Divides the dataset into smaller partitions to reduce the search space.
#### Scoring: 
Calculates the similarity between the query vector and the vectors in the selected partitions.
#### Rescoring: 
Refines the top-k results from the scoring phase to improve accuracy.

The ScaNN index is a tree-based quantization index for approximate nearest neighbor search. In Tree-quantization techniques, indexes learn a search tree together with a quantization (or hashing) function. When you run a query, the search tree is used to prune the search space while quantization is used to compress the index size. This pruning speeds up the scoring of the similarity (i.e., distance) between the query vector (user search text vector) and the database vectors. This results in increased efficiency of the nearest neighbor search. Read more about it [here]([url](https://cloud.google.com/alloydb/docs/ai/tune-indexes)).

   Refer to this blog for creating ScaNN index and executing Vector Search on indexed data: https://medium.com/google-cloud/upgrade-your-vector-search-efficiency-and-recall-with-scann-index-9bc8b2018377

### Install Extension:
#### CREATE EXTENSION IF NOT EXISTS alloydb_scann;

### Create Index (ScaNN):
#### CREATE INDEX toysearch_index ON toys
#### USING scann (text_embeddings cosine)
#### WITH (num_leaves=9); 

## Search Similar Toys: Gemini 2.0 for image based Vector Search
In this step, the app takes the user input (text / image) and asks Gemini for description to the level of detail that is comparable with the context in the toy descriptions  we have in the database. Then we match the embeddings and fetch the nearest neighbors.
For this step refer to the callGemini(String base64ImgWithPrefix) of the GeminiCall.java class.

## Create Your Toy: Imagen 3 Implementation
In this step, the app takes the user defined custom context for creating a new toy image and leverages Imagen 3 to accomplish this.
For this step, refer to the generateImage(String projectId, String location, String prompt) method of the generateToy class.

## More Like This
In this step, the app takes the image that the user clicked and finds similar toys to that. It also leverages the image based description matching method for vector search. Refer to the method: moreLikeThis(ModelMap map, Toy toy) in the Controller class.

## LangChain4j Integration
In this use case, we have used LangChain4j integration to implement the multimodal Gemini 2.0 invocation.

The goal of [LangChain4j]([url](https://docs.langchain4j.dev/intro)) is to simplify integrating LLMs into Java applications.

Here's how:

#### Unified APIs: 
LLM providers (like OpenAI or Google Vertex AI) and embedding (vector) stores (such as Pinecone or Milvus) use proprietary APIs. LangChain4j offers a unified API to avoid the need for learning and implementing specific APIs for each of them. To experiment with different LLMs or embedding stores, you can easily switch between them without the need to rewrite your code. LangChain4j currently supports 15+ popular LLM providers and 20+ embedding stores.
#### Comprehensive Toolbox:
While we are building numerous LLM-powered applications, identifying common abstractions, patterns, and techniques  is paramount. LangChain4j has refined these into a ready to use package. Their toolbox includes tools ranging from low-level prompt templating, chat memory management, and function calling to high-level patterns like Agents and RAG. For each abstraction, we provide an interface along with multiple ready-to-use implementations based on common techniques. 
### Numerous Examples: 
These examples showcase how to begin creating various LLM-powered applications, providing inspiration and enabling you to start building quickly.

## Toolbox Integration (Gen AI Toolbox for Databases):

Google's Open Source service that helps orchestrate between database and generative AI apps. Toolbox helps you build Gen AI tools that let your agents access data in your database. Toolbox provides:

Simplified development: Integrate tools to your agent in less than 10 lines of code, reuse tools between multiple agents or frameworks, and deploy new versions of tools more easily.
Better performance: Best practices such as connection pooling, authentication, and more.
Enhanced security: Integrated auth for more secure access to your data
End-to-end observability: Out of the box metrics and tracing with built-in support for OpenTelemetry.

Toolbox sits between your application's orchestration framework and your database, providing a control plane that is used to modify, distribute, or invoke tools. It simplifies the management of your tools by providing you with a centralized location to store and update tools, allowing you to share tools between agents and applications and update those tools without necessarily redeploying your application.

### Toolbox installation and details:

https://github.com/googleapis/genai-toolbox

https://pypi.org/project/toolbox-langchain/0.1.0/

https://github.com/googleapis/genai-toolbox-langchain-python

### tools.yaml 
This file (in this repo inside the toolbox folder) contains the tool implementation for this project to predict the price of the custom created toy by employing the Vector Search matches approach.


At the time of implementation, change the toolbox query to:
select avg(price) from (
  SELECT price FROM toys
      ORDER BY text_embeddings <=> CAST(embedding('text-embedding-005', 'elephant toys') AS vector(768))
      LIMIT 5
) as price  

---

## Build and Deploy
From your Cloud Shell Terminal, run the following commands:

  git clone https://github.com/AbiramiSukumaran/toysearch

Make sure to create the 2 Cloud Run Functions we are using in this project. The source code for those 2 are available in the folders get-toys-alloydb and toolbox-toys respectively. Once you create the Cloud Run Functions, delete these folders from your web app folder structure. 

  
  #### cd toysearch
  

  #### export PROJECT_ID YOUR_PROJECT_ID
  
  #### export GOOGLE_API_KEY YOUR_KEY
  

  #### mvn package
  
  #### mvn spring-boot:run
  
  
You should be able to run your app in your local cloud shell machine. 

## Serverless Deployment

#### gcloud run deploy --source .

Provide the necessary details as prompted and you should be able to get the app deployed serverlessly on Cloud Run.

