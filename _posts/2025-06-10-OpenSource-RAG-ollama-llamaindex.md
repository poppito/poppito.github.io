---
slug: "2025-08-08-OpenSource-RAG-with-llamaindex-ollama"
title:  "RAGs to riches (of information)"
date:   2025-08-08 06:00 +1100
categories: "blog"
---

# RAG (Retrieval augmented generation)

LLMs are great at generalised information, however retrieval augmented generation takes it a step further by localising the way LLMs return information by adding an extra layer of embeddings that they can call to answer questions or run searches.

In this guide, we'll build a local RAG system with 100% open source components and run it completely locally using [LlamaIndex](https://github.com/run-llama/llama_index) for document indexing and retrieval, [HuggingFace embeddings](https://huggingface.co/) for semantic search, and [Ollama](https://ollama.com/) to run the Llama3.2 model. To ensure we are working with reliable sources and we're referencing the data we've provided this model and that the model doesn't hallucinate, we'll also use citations.

---

## Prerequisites

- We'll be running our setup on a Mac with HomeBrew installed on your machine
- Python 3.9+
- Ollama installed and Llama3.2 model pulled
- Required Python libraries:
  ```bash
  pip install llama-index llama-index-llms-ollama llama-index-embeddings-huggingface torch transformers
  ```

## Ollama

Open up your terminal and run:

```
brew install ollama
```

Once installed, start the Ollama service:

```
brew services start ollama
```

## Llama 3.2

**Llama3.2** is Meta’s latest (currently, as of August 2025) and great for general-purpose coding and chat. Handles context well and doesn’t hallucinate as much as earlier versions.


To install Llama3.2:

```
ollama run llama3:2
```

This will pull the model if it doesn't already exist and then run it too.

## Setup

Your setup should look like this:

```markdown
RAG/
├── script.py
├── index_storage/
└── data/
    ├── document1.txt
    └── document2.pdf
```

The top level directory/folder can be whatever you want to call it. As long as you've installed the dependencies above, you can plonk the script below into the root of the folder. The `data` directory is where the script reads any text formatted data from to create the embeddings. Once the script is run, it will create `index_storage` where the embeddings are stored. 

## Code Breakdown
Below is the core script. It will:

- Use HuggingFace to create embeddings for your documents.
- Store the vector index on disk.
- Use Ollama's Llama3.2 model to answer queries.
- Show citations for each answer.


```python
import os
from llama_index.llms.ollama import Ollama
from llama_index.embeddings.huggingface import HuggingFaceEmbedding
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader, StorageContext, load_index_from_storage
from llama_index.core import Settings

# Suppress HuggingFace tokenizer parallelism warning
os.environ["TOKENIZERS_PARALLELISM"] = "false"

# Set up LLM and embedding model
Settings.llm = Ollama(model="llama3.2", request_timeout=30.0)
Settings.embed_model = HuggingFaceEmbedding(model_name="BAAI/bge-small-en-v1.5")

# Where we'll store embeddings
INDEX_DIR = "./index_storage"

# If the index doesn't exist create it then load it. It it does load it.
def persist_index():
    if os.path.exists(INDEX_DIR):
        storage_context = StorageContext.from_defaults(persist_dir=INDEX_DIR)
        index = load_index_from_storage(storage_context=storage_context)
        return index
    else:
        documents = []
        for filename in os.listdir("./data"):
            file_path = os.path.join("./data", filename)
            print("indexing " + file_path)
            if os.path.isfile(file_path):
                doc = SimpleDirectoryReader(input_files=[file_path]).load_data()
                for d in doc:
                    d.metadata = {"source": filename}
                documents.extend(doc)
        index = VectorStoreIndex.from_documents(documents=documents)
        index.storage_context.persist(persist_dir=INDEX_DIR)
        return index

# Start a loop to create a chat to query LLM for semantic search using RAG
# You should be able to adjust the similarity_top_k depending on how your embeddings perform
def run_query(index):
    query_engine = index.as_query_engine(similarity_top_k=5)
    print("Type your query or 'exit' to quit")
    while True:
        query = input("> ")
        if query.lower() == "exit":
            break
        response = query_engine.query(query)
        print("Response:")
        print(response.response)
        print("\nCitations:")
        if hasattr(response, "source_nodes") and response.source_nodes:
            for source in response.source_nodes:
                print(f"- {source.node.metadata.get('source', 'Unknown')}")
        else:
            print("No citations available.")

if __name__ == "__main__":
    index = persist_index()
    run_query(index=index)

```

## How It Works

- Creating Embeddings & Indexing
  The script loads all files from the data/ directory.
  Each file is read and converted into a document with metadata (the filename).
  Embeddings are created using the HuggingFace model.
  The vector index is persisted to disk in `./index_storage/`, so you don't need to re-embed every time. I could have made it more sophisticated by storing a hash of which files are indexed so every time the script is run, we check whether a new file is added or not, as currently you need to delete the entire embeddings directory whenever a new file is added.

- Querying with Llama3.2
  The script loads the index from disk (if it exists).
  You can type questions in the terminal.
  The system retrieves relevant documents using vector search and passes them to the Llama3.2 model running locally via Ollama.
  The answer is shown, along with citations (the filenames of the source documents).

## Example Usage

```bash
  $ python3 script.py
  indexing ./data/document1.txt #this can be any txt doc
  indexing ./data/document2.pdf #this can be any pdf doc
  Type your query or 'exit' to quit
  > What is the summary of document1?
  Response:
  [LLM-generated answer]

  Citations:
  - document1.txt
```


## Github Repo and preview
To see this in action, I've open sourced the code in this [github repository](https://github.com/poppito/ai-experiments/tree/main/rag) and used two of my favourite children's books from [Project Gutenberg](https://www.gutenberg.org)


With this setup, you have a fully local, privacy-preserving Q&A system over your own documents, powered by state-of-the-art open-source LLMs and embeddings. You can extend this further with a web or mobile UI, support for more file types, or advanced citation formatting.

## Preview

![](https://github.com/user-attachments/assets/9cedff4b-3c7b-4569-bec3-4c6184ae8e66)