---
title: Uploading the Vectors to Qdrant
---

Now that we have the embeddings, we can put them into Qdrant, allowing us to query the vectors without needing to redo the embeddings. 

1. Install the requirements for the upload code and start the Python REPL

```execute
pip install qdrant-client
python3
```

At this point, you should have startup records in the startups_demo.json file, encoded vectors in startup_vectors.npy and Qdrant running on a local machine.

Now you need to write a script to upload all startup data and vectors into the search engine.

2. Import the libraries and create a client connection to our running Qdrant database.

```execute
# Import client library
import numpy as np
import json
from qdrant_client import QdrantClient
from qdrant_client.models import VectorParams, Distance

qdrant_client = QdrantClient('http://localhost:6333')

```

3. Related vectors need to be added to a collection. Create a new collection for your startup vectors.

```execute 
qdrant_client.recreate_collection(
    collection_name='startups', 
    vectors_config=VectorParams(size=384, distance=Distance.COSINE),
)
```


{{< warning title="Info" >}}

* Use recreate_collection if you are experimenting and running the script several times. This function will first try to remove an existing collection with the same name.

* The vector_size parameter defines the size of the vectors for a specific collection. If their size is different, it is impossible to calculate the distance between them. 384 is the encoder output dimensionality. You can also use model.get_sentence_embedding_dimension() to get the dimensionality of the model you are using.

* The distance parameter lets you specify the function used to measure the distance between two points.

{{< /warning >}}

4. Create an iterator over the startup data and vectors.
   
   The Qdrant client library defines a special function that allows you to load datasets into the service. However, since there may be too much data to fit a single computer memory, the function takes an iterator over the data as input.

```execute
fd = open('./startups_demo.json')

# payload is now an iterator over startup data
payload = map(json.loads, fd)

# Load all vectors into memory, numpy array works as iterable for itself.
# Other option would be to use Mmap, if you don't want to load all data into RAM
vectors = np.load('./startup_vectors.npy')
```

5. Upload the data

```execute
qdrant_client.upload_collection(
    collection_name='startups',
    vectors=vectors,
    payload=payload,
    ids=None,  # Vector ids will be assigned automatically
    batch_size=256  # How many vectors will be uploaded in a single request?
)

execute()
```

Vectors are now uploaded and queryable in our Qdrant datastore. 