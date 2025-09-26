# RAG Implementation - Retrieval Augmented Generation

This document explains the Retrieval Augmented Generation (RAG) implementation for the chatbot.

```python
# app.py


import os
import chainlit as cl
from llama_index.core import (
    Settings,
    SimpleDirectoryReader,
    VectorStoreIndex,
    StorageContext,
    load_index_from_storage,
)
from llama_index.core.node_parser import SentenceWindowNodeParser
from llama_index.core.postprocessor import MetadataReplacementPostProcessor
from llama_index.postprocessor.cohere_rerank import CohereRank
from llama_index.llms.openrouter import OpenRouter
from llama_index.embeddings.openai import OpenAIEmbedding
from llama_index.core.callbacks import CallbackManager


# --- Configuration ---
# Ensure OPENAI_API_KEY, OPENROUTER_API_KEY, and COHERE_API_KEY are set in your environment
PERSIST_DIR = "./storage"
DATA_DIR = "./data"


# --- LlamaIndex Settings ---
# Note: OpenAI API key is used for embeddings in this example.
Settings.llm = OpenRouter(
    api_key=os.getenv("OPENROUTER_API_KEY"),
    model="anthropic/claude-3.5-sonnet",
    max_tokens=1024,
    context_window=8192,
)
Settings.embed_model = OpenAIEmbedding(
    model="text-embedding-3-large",
    api_key=os.getenv("OPENAI_API_KEY")
)
Settings.callback_manager = CallbackManager([cl.LlamaIndexCallbackHandler()])


@cl.on_chat_start
async def start():
    """
    Initializes the RAG engine at the start of a new chat session.
    Loads or creates the vector index and sets up the query engine.
    """
    if not os.path.exists(PERSIST_DIR):
        # Create the data directory if it doesn't exist
        if not os.path.exists(DATA_DIR):
            os.makedirs(DATA_DIR)
            # You can add a placeholder file if you want
            with open(os.path.join(DATA_DIR, "placeholder.txt"), "w") as f:
                f.write("Please add your documents to the 'data' directory.")
        
        # Load documents
        documents = SimpleDirectoryReader(DATA_DIR).load_data()
        
        # Create the sentence window node parser
        node_parser = SentenceWindowNodeParser.from_defaults(
            window_size=3,
            window_metadata_key="window",
            original_text_metadata_key="original_text",
        )
        nodes = node_parser.get_nodes_from_documents(documents)


        # Build and persist the index
        index = VectorStoreIndex(nodes)
        index.storage_context.persist(persist_dir=PERSIST_DIR)
    else:
        # Load the existing index
        storage_context = StorageContext.from_defaults(persist_dir=PERSIST_DIR)
        index = load_index_from_storage(storage_context)


    # Configure the re-ranker
    cohere_rerank = CohereRerank(
        api_key=os.getenv("COHERE_API_KEY"), 
        top_n=2
    )


    # Configure the metadata replacement post-processor
    postproc = MetadataReplacementPostProcessor(target_metadata_key="window")


    # Build the query engine
    query_engine = index.as_query_engine(
        similarity_top_k=10,
        node_postprocessors=[postproc, cohere_rerank],
        streaming=True,
    )


    cl.user_session.set("query_engine", query_engine)
    await cl.Message(
        content="Hello! I'm an advanced RAG assistant. How can I help you with your documents?"
    ).send()


@cl.on_message
async def main(message: cl.Message):
    """
    Handles incoming user messages, queries the RAG engine, and streams the response.
    """
    query_engine = cl.user_session.get("query_engine")
    response = await cl.make_async(query_engine.query)(message.content)


    msg = cl.Message(content="")
    await msg.send()


    for token in response.response_gen:
        await msg.stream_token(token)
    
    # Attach source nodes to the message for reference
    if response.source_nodes:
        source_elements = []
        for source_node in response.source_nodes:
            source_name = os.path.basename(source_node.metadata.get('file_name', 'Source'))
            source_elements.append(
                cl.Text(content=source_node.get_content(), name=source_name)
            )
        msg.elements = source_elements
        await msg.update()
    else:
        await msg.update()
```

## RAG System Overview

This implementation uses LlamaIndex to create a Retrieval Augmented Generation (RAG) system that can answer questions based on document content. The system includes:

1. **Document Processing**:
   - Loads documents from a specified directory
   - Parses documents into nodes using a sentence window approach
   - Creates and persists a vector index for efficient retrieval

2. **Advanced Retrieval**:
   - Uses OpenAI embeddings for semantic search
   - Implements Cohere re-ranking to improve result relevance
   - Applies sentence window context for better answer generation

3. **Response Generation**:
   - Uses OpenRouter's Claude 3.5 Sonnet model for high-quality responses
   - Streams tokens for a responsive user experience
   - Attaches source references to responses for transparency

4. **Chainlit Integration**:
   - Initializes the RAG engine at chat start
   - Processes user messages through the query engine
   - Displays source documents alongside responses

## Key Components

### 1. Document Indexing
The system creates a vector index from documents in the `./data` directory, using sentence windows to maintain context around individual chunks.

### 2. Query Processing
When a user asks a question:
1. The query is processed by the vector index to retrieve relevant nodes
2. Retrieved nodes are post-processed to expand context using sentence windows
3. Cohere re-ranking improves the relevance of results
4. The LLM generates a response based on the retrieved context

### 3. Source Attribution
The system attaches source documents to responses, allowing users to verify information and explore related content.

## Setup Requirements

- **API Keys**:
  - `OPENAI_API_KEY`: For embeddings
  - `OPENROUTER_API_KEY`: For LLM access
  - `COHERE_API_KEY`: For re-ranking

- **Directory Structure**:
  - `./data`: Place your documents here
  - `./storage`: Vector index storage location

## Usage

1. Add documents to the `./data` directory
2. Start the Chainlit application
3. Ask questions about your documents
4. View sources attached to each response
