---
title: Prepare the Sample Dataset
---

To conduct a neural search on startup descriptions, you must first encode the description data into vectors. To process text, you can use a pre-trained models like [BERT](https://en.wikipedia.org/wiki/BERT_(language_model)) or sentence transformers. The [sentence-transformers](https://github.com/UKPLab/sentence-transformers) library lets you conveniently download and use many pre-trained models, such as DistilBERT, MPNet, etc.

We have already downloaded the dataset and saved the file in your environment as `startups_demo.json`

1. The first step is going to be `pip install` our code requirements. These libraries are quite large and may take a while to download. 

```execute
pip install sentence-transformers numpy pandas tqdm
```
_If you click on the little running man in the blue bar the code block will execute in the terminal window_

2. Start up the Python REPL

```execute
python3 
```

3. Import all relevant models.

```execute
from sentence_transformers import SentenceTransformer
import numpy as np
import json
import pandas as pd
from tqdm.notebook import tqdm
```

4. You will be using a pre-trained model called _all-MiniLM-L6-v2_. This is a performance-optimized sentence embedding model and you can read more about it and other available models [here](https://www.sbert.net/docs/pretrained_models.html).

Download and create our pre-trained sentence encoder. 

```execute
model = SentenceTransformer('all-MiniLM-L6-v2', device="cpu")
```

5. Read the raw data file, `startups_demo.json`

```execute 
df = pd.read_json('./startups_demo.json', lines=True)
```

6.  Encode all startup descriptions to create an embedding vector for each observation. Internally, the encode function will split the input into batches, which will significantly speed up the encoding process.

Since we are using the CPU for encoding, this can take up to 10 minutes.

```execute
vectors = model.encode([
    row.alt + ". " + row.description
    for row in df.itertuples()
], show_progress_bar=True)
```

All of the descriptions are now converted into vectors. There are 40474 vectors of 384 dimensions which corresponds to the  output layer of the model.

```execute
vectors.shape
```

7. Save the vectors to a new file named startup_vectors.npy, which is a serialized NumPy array. After that we exit the Python REPL.

```execute
np.save('startup_vectors.npy', vectors, allow_pickle=False)

exit()
```


