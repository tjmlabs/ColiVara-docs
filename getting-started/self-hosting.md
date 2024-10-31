---
icon: globe-pointer
---

# Self-hosting

ColiVara is made up of multiple services. The first is an embedding service that turns images and queries into Vectors. This is bundled separately as it needs a GPU. The second is [a Postgres with a pgvector](https://github.com/pgvector/pgvector) extension to store vectors. A [Gotenberg](https://gotenberg.dev/) service that handles document conversions to PDFs. And finally, a [Django-Ninja](https://django-ninja.dev/) REST API that handles user requests. Other than the Embedding service, everything else is bundles together via docker-compose to run seamlessly on a typical VPS.

For production workloads - you may consider a managed Postgres instance for automatic security updates and regular backup.&#x20;

### Embedding Service&#x20;

1. Git clone the service repository&#x20;

```bash
git clone https://github.com/tjmlabs/ColiVarE
```

2. Optional: download uv and install it in your environment. We use uv, however you can also use pip to install the requirements.

```bash
pip install uv
uv venv # or python -m venv .venv
source .venv/bin/activate #.venv/Scripts/activate on windows
```

3. Compile requirements based on your environment. As this services uses pytorch under the hood- requirements will be different depending on your OS and Nvidia GPU availability. We use a mac for development and a Linux in production.
4. Install the requirements&#x20;

```bash
uv pip compile builder/requirements.in -o builder/requirements.txt
```

5. Download the models from huggingface and save them in the `models_hub` directory before building. See src/download\_models.py for more details.

```python
from colpali_engine.models import ColQwen2, ColQwen2Processor
import torch

model_name = "vidore/colqwen2-v0.1"
if torch.cuda.is_available():
    device_map = "cuda"
elif torch.backends.mps.is_available():
    device_map = "mps"
else:
    device_map = None
    
model = ColQwen2.from_pretrained(
        model_name,
        cache_dir="models_hub/",  # where to save the model
        device_map=device_map,
    )

processor = ColQwen2Processor.from_pretrained(model_name, cache_dir="models_hub/")
```

6. Run the service locally using the following command

```bash
python3 src/handler.py --rp_serve_api
```

7. The Embedding service is now running on `http://localhost:8000/`. You can test it using the following command. Remember - you do need a GPU and at least 8gb of VRAM available. The performance on a M-series of Macs is also acceptable for local development.&#x20;

```bash
curl --request POST \
  --url http://localhost:8000/runsync \
  --header 'Content-Type: application/json' \
  --data '{"input": {"task": "query","input_data": ["hello"]}}'  
```

You may consider running this service in an "on-demand" fashion via Docker for cost-savings in production settings.&#x20;

### REST API

1. Clone the ColiVara repository&#x20;

```bash
git clone https://github.com/tjmlabs/ColiVara
```

2. Create a .env.dev file in the root directory with the following variables:

```
EMBEDDINGS_URL="the serverless embeddings service url" # for local setup use http://localhost:8000/runsync/
EMBEDDINGS_URL_TOKEN="the serverless embeddings service token"  # for local setup use any string will do.
```

3. Run all the services via docker-compose

````bash
``bash
docker-compose up -d --build
docker-compose exec web python manage.py migrate
docker-compose exec web python manage.py createsuperuser
# get the token from the superuser creation
docker-compose exec web python manage.py shell
from accounts.models import CustomUser
user = CustomUser.objects.first().token # save this token
```
````

4. Application will be running at http://localhost:8001 and the swagger documentation at `http://localhost:8001/v1/docs`
5. The swagger documentations page is also a playground - where you can try all the endpoints using the token created earlier

<figure><img src="../.gitbook/assets/Screenshot 2024-10-21 at 9.45.11 AM.png" alt=""><figcaption></figcaption></figure>

### Development

Follow the steps above to get the service up and running.&#x20;

1.  To run tests and type checking - we have 100% test coverage

    ```bash
    docker-compose exec web pytest 
    #mypy for type checking
    docker-compose exec web mypy .
    ```
2. Make a branch with your changes and additional code&#x20;
3. Open a Pull request on Github. We have CI/CD and pre-commit hooks to format and test your changes&#x20;

We welcome contribution and discussion.&#x20;
