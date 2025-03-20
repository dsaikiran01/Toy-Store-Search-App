# Toy Store Search App

Welcome to the **Toy Store Search App** repository! This application leverages advanced AI capabilities and cloud technologies to provide a seamless and personalized toy shopping experience. Users can search for toys using text or images, and even design their own custom creations. 

The backend of the application is built using **Spring Boot**, and the **Gen AI Toolbox for Databases** is implemented in **Python** to facilitate AI-powered database interactions on **Google Cloud**.

The app utilizes **AlloyDB**, **pgvector**, **Gemini 2.0 Flash**, **Imagen 3**, and the **Gen AI Toolbox for Databases** to deliver these features.

> **This project was developed as part of Google Cloud's Code Vipassana Season 9**  

## Architecture

The application architecture consists of the following components:

1. **AlloyDB**: A fully managed, PostgreSQL-compatible database that stores toy data, including descriptions, images, and attributes.

2. **pgvector**: A PostgreSQL extension that enables storage and similarity search of vector embeddings, facilitating semantic search capabilities.

3. **Gemini 2.0 Flash**: An AI model used to analyze user-uploaded images and extract contextual information for accurate search results.

4. **Imagen 3**: A generative AI model that allows users to create custom toy designs based on text prompts.

5. **Gen AI Toolbox for Databases**: An open-source server that simplifies the development of AI tools interacting with databases, enhancing performance, security, and observability.


## Setup and Installation

### 1. Set Up AlloyDB Database

**AlloyDB** is a PostgreSQL-compatible database that offers high performance and scalability. Follow these steps to set up the database:

- **Create a Cluster and Instance**: Navigate to the AlloyDB page in the Google Cloud Console and create a cluster named `vector-cluster` and an instance named `vector-instance`. Detailed instructions can be found in the [Toy Store Search App Codelab][codelab-toy-store-app].

- **Create the `toys` Table**:

  ```sql
  CREATE TABLE toys (
    id VARCHAR(25),
    name VARCHAR(25),
    description VARCHAR(20000),
    quantity INT,
    price FLOAT,
    image_url VARCHAR(200),
    text_embeddings vector(768)
  );
  ```

- **Insert Data**: Populate the `toys` table with data. Sample insert scripts are available in the `data.sql` file in this repository.

- **Enable Extensions**:

  ```sql
  CREATE EXTENSION vector;
  CREATE EXTENSION google_ml_integration;
  ```

- **Grant Permissions**:

  ```sql
  GRANT EXECUTE ON FUNCTION embedding TO postgres;
  ```

- **Grant Vertex AI User Role**:

  ```bash
  PROJECT_ID=$(gcloud config get-value project)
  gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:service-$(gcloud projects describe $PROJECT_ID --format="value(projectNumber)")@gcp-sa-alloydb.iam.gserviceaccount.com" \
    --role="roles/aiplatform.user"
  ```

- **Update Text Embeddings**:

  ```sql
  UPDATE toys SET text_embeddings = embedding('text-embedding-005', description);
  ```

- **Create ScaNN Index**:

  ```sql
  CREATE EXTENSION IF NOT EXISTS alloydb_scann;
  CREATE INDEX toysearch_index ON toys
  USING scann (text_embeddings cosine)
  WITH (num_leaves=9);
  ```

### 2. Install and Configure Gen AI Toolbox

The **Gen AI Toolbox for Databases** simplifies the integration of AI tools with databases. Follow these steps to install and configure the toolbox:

- **Clone the Toolbox Repository**:

  ```bash
  git clone https://github.com/googleapis/genai-toolbox
  cd genai-toolbox
  ```

- **Install Dependencies**:

  ```bash
  pip install -r requirements.txt
  ```

- **Configure Tools**: Define your tools in the `tools.yaml` file. For example, to create a price prediction tool:

  ```yaml
  tools:
    - name: price_prediction
      description: Predicts the price of a toy based on its description.
      source: alloydb
      parameters:
        query: |
          SELECT AVG(price) FROM (
            SELECT price FROM toys
            ORDER BY text_embeddings <=> CAST(embedding('text-embedding-005', '{{description}}') AS vector(768))
            LIMIT 5
          ) AS price;
  ```

- **Deploy Toolbox on Cloud Run**:

  ```bash
  gcloud run deploy toolbox-service --source .
  ```

  Detailed instructions are available in the [Gen AI Toolbox Codelab][codelab-genai-toolbox].

### 3. Build and Deploy the Application

- **Clone the Application Repository**:

  ```bash
  git clone https://github.com/your-username/toy-store-search-app.git
  cd toy-store-search-app
  ```

- **Set Environment Variables**:

  ```bash
  export PROJECT_ID=your_project_id
  export GOOGLE_API_KEY=your_api_key
  ```

- **Build and Run Locally**:

  ```bash
  mvn package
  mvn spring-boot:run
  ```

- **Deploy on Cloud Run**:

  ```bash
  gcloud run deploy --source .
  ```

Below is a screenshot of the deployed app:

![Deployed Toy Store Search App](deployed_screenshot.png)

## Features

### 1. Contextual Search with AI-Powered RAG

Utilizing **AlloyDB** and **pgvector**, the application performs semantic searches based on user input. By generating vector embeddings of toy descriptions and user queries, it identifies the most relevant matches using cosine similarity and ScaNN indexing.

### 2. Image-Based Search with Gemini 2.0 Flash

Users can upload images of toys, and the application employs **Gemini 2.0 Flash** to extract descriptive features. These features are then used to perform a vector search, retrieving toys that closely match the uploaded image.

### 3. Custom Toy Creation with Imagen 3

The application allows users to design custom toys by providing text descriptions. **Imagen 3** generates images based on these descriptions, enabling users to visualize and create their desired toys.

### 4. Price Prediction Powered by Gen AI Toolbox

By integrating the **Gen AI Toolbox for Databases**, the application predicts the price of custom-created toys. It analyzes similar existing toys in the database to provide an estimated price for the new creation.


## Resources

Here are some useful links and references to help you understand and set up the project:

1. [Toy Store Search App with Cloud Databases, Serverless Runtimes and Open Source Integrations
](https://codelabs.developers.google.com/toy-store-app#0)  

2. [Video guide for application setup on Google Cloud](https://www.youtube.com/watch?v=pk6wb_sOWlM)

3. [Installing and Setting-up Toolbox for your Gen AI & Agentic Applications on AlloyDB
](https://codelabs.developers.google.com/genai-toolbox-for-alloydb#0)

4. [Video guide for Toolbox setup on Google Cloud](https://www.youtube.com/watch?v=EEe_7jsEWV8)
