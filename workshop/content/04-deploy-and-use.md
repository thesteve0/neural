---
title: Deploy and Use Our Application
---

https://qdrant.tech/documentation/tutorials/neural-search/#deploy-the-search-with-fastapi

## Add Search Code to API

Time to put our search code into a web API

1. PIP install FastAPI and uvicorn.

```execute
pip install fastapi uvicorn
```

2. Implement the service.


The service will have a single API endpoint for search. Open a new file, titled "service.py" in the integrated editor and add the following code: 

```editor:append-lines-to-file
file: ~/service.py
text: |

    from fastapi import FastAPI

    # The file where NeuralSearcher is stored
    from neural_searcher import NeuralSearcher

    app = FastAPI()

    # Create a neural searcher instance
    neural_searcher = NeuralSearcher(collection_name='startups')

    @app.get("/api/search")
    def search_startup(q: str):
        return {
            "result": neural_searcher.search(text=q)
        }


    if __name__ == "__main__":
        import uvicorn
        uvicorn.run(app, host="127.0.0.1", port=8000)
```

3. Save the file

```editor:execute-command
command: workbench.action.files.saveFiles
```

4. Go back to the terminal and start the service:

```execute
python3 service.py
```
5. Open a web browser and start searching!

Feel free to play around with it, make queries regarding the companies in our corpus, and check out the results.


```dashboard:create-dashboard
name: Search
url: {{< param ingress_protocol >}}://search-{{< param session_name >}}.{{< param ingress_domain >}}/docs
```



