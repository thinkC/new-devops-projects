# Python App in Docker
In this project we are going to create a CRUD python flask application in docker.

## Step 1: 

Create Project directory

```bash
mkdir oxygen-python-docker-app
cd oxygen-python-docker-app
mkdir product-service
cd product-service
python3 -m venv venv or python -m venv venv
```

Activate Virtual environment 

```bash
#on linux
# Still in the product-service directory
source venv\bin\activate

#on windows
.\venv\Scripts\Activate.ps1
```
Install Flask

```bash
# ensure pip is already installed
pip list   # check list of installed app or pip3 list

# install flask
pip install Flask
```


Create a src folder in the _product-service_ folder

```bash
mkdir src
cd src
touch app.py or New-Item app.py # if using powershell
```
Add the code below to _app.py_

```python
from flask import Flask, jsonify, request

products = [
    {"id": 1, "name":"Product 1"},
    {"id": 2, "name":"Product 2"}
]

app = Flask(__name__)

@app.route("/products")
def get_products():
    return jsonify(products)

@app.route('/product/<int:id>')
def get_product(id):
    product_list = [product for product in products if product['id'] == id]
    if len(product_list) == 0:
        return f'Product with id {id} not found', 404
    return jsonify(product_list[0])


@app.route('/product', methods=['POST'])
def post_product():
    # Retrieve the product from the request body
    request_product = request.json

    # Generate an ID for the post
    new_id = max([product['id'] for product in products]) + 1

    # Create a new product
    new_product = {
        'id': new_id,
        'name': request_product['name']
    }

    # Append the new product to our product list
    products.append(new_product)

    # Return the new product back to the client
    return jsonify(new_product), 201



@app.route('/product/<int:id>', methods=['PUT'])
def put_product(id):
    # Get the request payload
    updated_product = request.json

    # Find the product with the specified ID
    for product in products:
        if product['id'] == id:
            # Update the product name
            product['name'] = updated_product['name']
            return jsonify(products), 200

    return f'Product with id {id} not found', 404


# curl --request DELETE -v http://localhost:5000/product/2
@app.route('/product/<int:id>', methods=['DELETE'])
def delete_product(id):
    # Find the product with the specified ID
    product_list = [product for product in products if product['id'] == id]
    if len(product_list) == 1:
        products.remove(product_list[0])
        return f'Product with id {id} deleted', 200

    return f'Product with id {id} not found', 404

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0')
```

Start Flask

```bash
#Inside product-service folder

python .\src\app.py # or python3 .\src\app.py
```

Test the endpoint on the browser or on Postman or curl

Output

```bash
C:\Users\user1>curl http://127.0.0.1:5000/products
[
  {
    "id": 1,
    "name": "Product 1"
  },
  {
    "id": 2,
    "name": "Product 2"
  }
]


```
Get On product

```bash
C:\Users\user1>curl http://127.0.0.1:5000/product/1
{
  "id": 1,
  "name": "Product 1"
}

```
For product that does not exist

```bash
C:\Users\user1>curl http://127.0.0.1:5000/product/100
Product with id 100 not found
```
Create new Product in Postman

![create-new-product](https://github.com/thinkC/new-devops-projects/blob/master/parent-img/oxygen-python-docker-app/img1.png?raw=true)


Check

```bash
C:\Users\user1>curl http://127.0.0.1:5000/products
[
  {
    "id": 1,
    "name": "Product 1"
  },
  {
    "id": 2,
    "name": "Product 2"
  },
  {
    "id": 3,
    "name": "Product 3"
  }
]

```

Update Product 

![update-product](https://github.com/thinkC/new-devops-projects/blob/master/parent-img/oxygen-python-docker-app/img2.png?raw=true)

check

```bash
C:\Users\user1>curl http://127.0.0.1:5000/products
[
  {
    "id": 1,
    "name": "Product 1"
  },
  {
    "id": 2,
    "name": "updated Product"
  },
  {
    "id": 3,
    "name": "Product 3"
  }
]
```
Delete Product

![delete-product](https://github.com/thinkC/new-devops-projects/blob/master/parent-img/oxygen-python-docker-app/img3.png?raw=true)

Check

```bash

C:\Users\user1>curl http://127.0.0.1:5000/products
[
  {
    "id": 1,
    "name": "Product 1"
  },
  {
    "id": 2,
    "name": "updated Product"
  }
]

```

## Step 2: Dockerizing  Flask Application

Inside product-service folder create a `Dockerfile`

```bash
touch Dockerfile
touch requirements.txt
```

Add the code below to the _Dockerfile_

```dockerfile
#set base image (host OS)
FROM python:3.9-alpine

#Set the working directory in the container
WORKDIR /code

#Copy the dependencies file to the working directory
COPY requirements.txt .

# Install dependencies
RUN pip install -r requirements.txt

#copy the content of the local src directory to the working directory
COPY src/ .

# command to run on the containmer start
CMD ["python", "./app.py"]

```

Add below to _requirements.txt_
```python
Flask==2.3.3
```

Run docker command to build a image in the current directory

```bash
docker build -t productservice .
```

Run below to see the built image

```bash
(venv) PS C:\Users\user1\Documents\Tutorial\oxygen-python-docker-app\product-service> docker images
REPOSITORY                                                                  TAG
          IMAGE ID       CREATED          SIZE
productservice                                                              latest
          60bc72a50825   12 seconds ago   55MB
```

Start the container from the built image. The '-d' run it in the background, '-p' - the first '5000' port the application would run on on the host while the other '5000' is the port the application would run in the container. `productservice` is the name of the image

```bash
docker run -d -p 5000:5000 productservice
```

output

```bash
(venv) PS C:\Users\user1\Documents\Tutorial\oxygen-python-docker-app\product-service> docker run -d -p 5000:5000 productservice
c14124aa3ee8e3d5c19cc42475a54acb671f1edd9fd41c6999e4815b2fc4105d
```

```bash
docker rm 5398d835eb07 # To delete a stopped container where '5398d835eb07' is the container ID
docker rmi 60bc72a50825 # To delete an image where '60bc72a50825'' is teh container ID
```

Run docker ps to see the running container

```bash
(venv) PS C:\Users\user1\Documents\Tutorial\oxygen-python-docker-app\product-service> docker ps
CONTAINER ID   IMAGE            COMMAND             CREATED         STATUS         PORTS                    NAMES
aafd0f5eb8a9   productservice   "python ./app.py"   3 seconds ago   Up 3 seconds   0.0.0.0:5000->5000/tcp   compassionate_bardeen
```

Test the app to get list of products, add new product and other CRUD operations

```bash
C:\Users\user1>curl http://127.0.0.1:5000/products
[
  {
    "id": 1,
    "name": "Product 1"
  },
  {
    "id": 2,
    "name": "Product 2"
  }
]
```
Login to the docker container 

```bash
docker exec -it <container id> bash # if you are using full image and not alpine
docker exec -it aafd0f5eb8a9 ash # if you are using alpine image
```

output

```bash
(venv) PS C:\Users\user1\Documents\Tutorial\oxygen-python-docker-app\product-service> docker exec -it aafd0f5eb8a9 ash
/code # ls -la
total 16
drwxr-xr-x    1 root     root          4096 Oct 31 16:30 .
drwxr-xr-x    1 root     root          4096 Oct 31 16:48 ..
-rwxr-xr-x    1 root     root          2267 Oct 31 14:17 app.py
-rwxr-xr-x    1 root     root            12 Oct 31 16:45 requirements.txt
```

Stop the running container


```bash
docker stop <container ID>
```

## Step 3: Running Multiple Containers with Docker Compose

