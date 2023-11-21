# Definition 
- **Actor** is the self-contained unit that wrapped the code to receive messages and process concurrency 
- A large number of actors can execute simultaneously, and actors execute independently from each other.

# Run Dapr Actor 
```shell 
dapr init --slim

cp application/components/* ~/.dapr/components/

~/.dapr/bin/placement


dapr run --app-id embedding-actor --app-port 3000 -- uvicorn --port 3000 python.sdk.actor.openai_embedding_actor.openai_embedding_actor_service:app

dapr run --app-id apptest python python/sdk/actor/openai_embedding_actor/client.py
```
