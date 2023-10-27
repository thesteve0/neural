---
title: Building the Search API
---



Now that all the preparations are complete, let’s start building a neural search API that reads from our Qdrant database.

In order to process incoming requests, neural search will need 2 things: 
1) Using our embedding model to convert the query into a vector in the same coordinate space as our data vectors
2) Qdrant client to perform search queries using the generated query embedding.


## Search Code

Let's write our search code. 

1. Open a new file, titled "neural_searcher.py" in the integrated editor and add the following code:

```editor:append-lines-to-file
file: ~/neural_searcher.py
text: |
    from qdrant_client import QdrantClient
    from qdrant_client.models import Filter
    from sentence_transformers import SentenceTransformer


    class NeuralSearcher:

        def __init__(self, collection_name):
            self.collection_name = collection_name
            # Initialize encoder model
            self.model = SentenceTransformer('all-MiniLM-L6-v2', device='cpu')
            # initialize Qdrant client
            self.qdrant_client = QdrantClient('http://localhost:6333')

```

2. Now that the imports and constructor are coded, let's add a search function:

```editor:append-lines-to-file
file: ~/neural_searcher.py
text: |4

        def search(self, text: str):
            # Convert text query into vector
            vector = self.model.encode(text).tolist()

            # Use `vector` for search for closest vectors in the collection
            search_result = self.qdrant_client.search(
                collection_name=self.collection_name,
                query_vector=vector,
                query_filter=None,  # If you don't want any filters for now
                limit=5  # 5 the most closest results is enough
            )
            # `search_result` contains found vector ids with similarity scores along with the stored payload
            # In this function you are interested in payload only
            payloads = [hit.payload for hit in search_result]
            return payloads
```
3. With Qdrant it is also feasible to add some payload conditions to the search, these are implemented through Filters. Let's add a filter to allow the query to restrict results to certain cities. In our example below we hard code in "Berlin" but making City name part of the API would be a good exercise:

{{< warning title="ERROR" >}}
Steve says: This code would never get exercised - not sure why it is written this way. Should be a different search method and then our API should offer both. 

I am making it non-actionable for now
{{< /warning >}}

```
editor:append-lines-to-file
file: ~/neural_searcher.py
text: |

        city_of_interest = "Berlin"

        # Define a filter for cities
        city_filter = Filter(**{
            "must": [{
                "key": "city", # Store city information in a field of the same name 
                "match": { # This condition checks if payload field has the requested value
                    "value": "city_of_interest"
                }
            }]
        })

        search_result = self.qdrant_client.search(
            collection_name=self.collection_name,
            query_vector=vector,
            query_filter=city_filter,
            limit=5
        )

```

Go ahead and save the file.

```editor:execute-command
command: workbench.action.files.saveFiles
```

