# FLOOWBOX - Semantic Recipe Recommendations (Qdrant + Mistral)

Built this to learn Qdrant properly before using it in more complex client workflows. Recipe search was a clean domain to test the full RAG loop.

## What it does

Embeds recipe data into Qdrant as vectors. Takes a user query (ingredients, cuisine, dietary preference), runs semantic similarity search against the vector store, and uses Mistral to generate a natural recommendation response with the most relevant recipes.

## Tools Used
- **Orchestration:** n8n
- **Vector DB:** Qdrant
- **Embeddings + LLM:** Mistral
- **Logic:** Semantic similarity search

## Why I built this
I needed hands-on experience with Qdrant before deploying it for a client's customer support RAG system. This was my learning project — kept it simple enough to understand every step.

## What I learned from this
The chunking strategy matters more than I expected. Recipe ingredients vs instructions need to be embedded differently to get good retrieval results.
