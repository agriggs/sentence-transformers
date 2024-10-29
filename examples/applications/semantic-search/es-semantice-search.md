# Example of keyword and semantic search with Elasticsearch

```
python -m venv .venv
pip install -r requirements.txt
```

Linux
```
export PYTHONPATH=$PYTHONPATH:/home/aarongriggs/dev/sentence-transformers
```

Windows PowerShell
```
$env:PythonPath = "C:\Users\agriggs\dev\sentence-transformers"
```

Windows Command
```
set PYTHONPATH=%PYTHONPATH%;C:\Users\agriggs\dev\sentence-transformers
```

cd to directory where file exist and run
```
cd examples\applications\semantic-search
python semantic_search_quora_elasticsearch.py
```

Code is [here](./semantic_search_quora_elasticsearch.py)
```
es = Elasticsearch(
    hosts=["https://localhost:9200"],
    basic_auth=("elastic", os.environ["ELASTIC_PASSWORD"]),  # displayed at ES server startup
    ssl_context=create_default_context(cafile="../http_ca.crt"),  # copied from inside ES container
)
```

## Model
```
model = SentenceTransformer("quora-distilbert-multilingual")
```
## ES index mappings
```
# TODO: Update ES password and cafile to use env file (_env.py)
es = Elasticsearch(
    hosts=["https://localhost:9200"],
    basic_auth=("elastic", os.environ["ELASTIC_PASSWORD"]),  # displayed at ES server startup
    ssl_context=create_default_context(cafile="C:/Users/agriggs/dev/http_ca.crt"),  # copied from inside ES container
)
```

## Encoding of the vector and adding to the ES index
```
for start_idx in range(0, len(qids), chunk_size):
    end_idx = start_idx + chunk_size

    embeddings = model.encode(questions[start_idx:end_idx])
    bulk_data = []
    for qid, question, embedding in zip(qids[start_idx:end_idx], questions[start_idx:end_idx], embeddings):
        bulk_data.append(
            {
                "_index": "quora",
                "_id": qid,
                "_source": {"question": question, "question_vector": embedding},
            }
        )

    helpers.bulk(es, bulk_data)  
```

## Search by keyword and semantic
```
question_embedding = model.encode(inp_question)
   
# Lexical search
bm25 = es.search(index="quora", body={"query": {"match": {"question": inp_question}}})

# Semantic search
sem_search = es.search(
    index="quora",
    knn={"field": "question_vector", "query_vector": question_embedding, "k": 10, "num_candidates": 100},
)
```