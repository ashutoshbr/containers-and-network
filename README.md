## 1\. Create a network 🕸️

```bash
docker network create my-net
```

Always replace `my-net` with a network name of your choice.

## 2\. Setup containers🚀

### Database (postgres)

Pull the latest image from the docker hub.

```bash
docker pull postgres
```

Running the image as a container.

```bash
docker run -dit --network=my-net --name=db-container -e POSTGRES_PASSWORD=abc123 postgres
```

Replace:

1\. `db-container` with the database container name of your choice.

2\. `abc123` with a password for postgres.

*Enter dummy data into the database*

```sql
CREATE TABLE COMPANY(
   ID INT PRIMARY KEY     NOT NULL,
   NAME           TEXT    NOT NULL,
   AGE            INT     NOT NULL
);
```

```sql
INSERT INTO COMPANY (ID,NAME,AGE) VALUES (1, 'Nick', 29);
```

### Server (FastAPI)

#### Server code

###### [main.py](http://main.py)

```python
# main.py
from fastapi import FastAPI
from .database import conn, cursor

app = FastAPI()

@app.get("/")
def home():
    return {"msg": "Hello"}

@app.get("/company")
def get_company():
    cursor.execute(""" SELECT * FROM company """)
    company = cursor.fetchall()
    return company
```

Execute the SQL query of your choice as per the available data in the postgres database.

###### [database.py](http://database.py)

```python
# database.py
import psycopg

# conn = psycopg.connect("postgresql://postgres:abr123@my_net:5432")
conn = psycopg.connect("dbname='postgres' user='postgres' host='db' password='abc123'")
cursor = conn.cursor()
```

Here:  
`dbname='postgres'` & `user='postgres'`: postgres is the default database self-created  
`host='db'`: **db** is the database container name on the same network  
Use your password instead of `abc123`

###### requirements.txt

```python
# requirements.txt
anyio==3.6.2
click==8.1.3
fastapi==0.95.1
h11==0.14.0
idna==3.4
psycopg==3.1.9
psycopg-binary==3.1.9
psycopg-pool==3.1.7
pydantic==1.10.7
sniffio==1.3.0
starlette==0.26.1
typing_extensions==4.5.0
uvicorn==0.22.0
```

###### Dockerfile

Build the docker image from the `Dockerfile`.

```docker
# FROM python:3.11
FROM python:3.11.4-alpine3.18

# 
WORKDIR /code

# 
COPY ./requirements.txt /code/requirements.txt

# 
RUN pip install --no-cache-dir --upgrade -r /code/requirements.txt

# 
COPY ./app /code/app

# 
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "80"]
```

###### Build and Run the docker container

```bash
docker build -t server-image .
```

Replace `server-image` with server image name of your choice.

Running the image as a container.

```bash
docker run -dit --network=my-net --name=server-container server-image
```

Replace:  
1\. `server-container` with server container name of your choice.  
2\. `server-image` with server name you choose.

## 3\. Ensure both the containers are in the same name network after they are run

```bash
docker network inspect my-net
```

## Extras

Check logs of a container

```bash
docker logs container-id
```

Get shell access to a container

```bash
docker exec -it container-id bash
```

GET request using curl

```bash
curl 127.0.0.1:8000/company
```
