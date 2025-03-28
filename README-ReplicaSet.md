
# MongoDB Replica Set - Guia Completo de Configuração e Teste

Este projeto demonstra passo a passo como configurar um **Replica Set** do MongoDB utilizando **Docker**, além de testar sua alta disponibilidade simulando a queda e o restabelecimento de um nó.

---

## 📌 Passo 1 - Criação da Docker Network

Crie a rede Docker para o cluster:

```bash
docker network create mongoCluster
```

Se receber o erro:

```
Error response from daemon: network with name mongoCluster already exists
```

Verifique a existência da rede:

```bash
docker network ls
```

Caso exista, remova:

```bash
docker network rm mongoCluster
```

Depois, crie novamente:

```bash
docker network create mongoCluster
```

---

## 📌 Passo 2 - Iniciar Instâncias do MongoDB

Inicie os contêineres do MongoDB:

### Nó 1

```bash
docker run -d --rm -p 27017:27017 --name mongo10 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo10
```

### Nó 2

```bash
docker run -d --rm -p 27018:27017 --name mongo20 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo20
```

### Nó 3

```bash
docker run -d --rm -p 27019:27017 --name mongo30 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo30
```

---

## 📌 Passo 3 - Configurar o Replica Set

Acesse o contêiner `mongo10`:

```bash
docker exec -it mongo10 mongosh
```

Verifique se está conectado:

```js
db.runCommand({hello:1})
```

Inicie o Replica Set:

```js
rs.initiate({
  _id: "myReplicaSet",
  members: [
    {_id: 0, host: "mongo10"},
    {_id: 1, host: "mongo20"},
    {_id: 3, host: "mongo30"}
  ]
})
```

Saia do contêiner:

```bash
exit
```

---

## 📌 Passo 4 - Verificar o Replica Set

Verifique o estado do cluster:

```bash
docker exec -it mongo10 mongosh --eval "rs.status()"
```

---

## 📌 Passo 5 - Conectar via MongoDB Compass

Utilize a URL de conexão:

```
mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.3.8
```

Caso ao conectar não apareça `[direct: primary]`, ajuste para a porta do primário (exemplo):

```
mongodb://127.0.0.1:27019/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.3.8
```

Para saber qual é o primário:

```js
rs.isMaster().primary
```

---

## 📌 Passo 6 - Inserção de Dados

No shell, insira registros:

```js
use CorporeSystem

db.cliente.insertOne({codigo: 1, nome: "Ana Maria"});
db.cliente.insertOne({codigo: 2, nome: "Maria Jose"});
db.cliente.insertOne({codigo: 3, nome: "Jose Silva"});
db.cliente.insertOne({codigo: 4, nome: "Luis Souza"});
db.cliente.insertOne({codigo: 5, nome: "Fernanda Silva"});
```

Verifique:

```js
db.cliente.find()
```

---

## 📌 Passo 7 - Testando Alta Disponibilidade

### Simular queda do nó primário:

```bash
docker stop mongo10
```

Verificar o estado:

```bash
docker exec -it mongo20 mongosh --eval "rs.status()"
```

Tente inserir dados:

```js
db.cliente.insertOne({codigo: 6, nome: "Teresa Maria"});
```

Abra uma nova conexão apontando para o novo primário:

```
mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.3.8
```

Consultar e inserir dados:

```js
use CorporeSystem

db.cliente.findOne()

db.cliente.insertOne({codigo: 7, nome: "Joao Cardoso"});
```

---

## 📌 Passo 8 - Restabelecendo o nó

Voltar com o nó `mongo10`:

```bash
docker run -d --rm -p 27017:27017 --name mongo10 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo10
```

Verificar estado do cluster:

```bash
docker exec -it mongo20 mongosh --eval "rs.status()"
```

O nó `mongo20` estará como secundário.

---

## ✅ Conclusão

Este exercício demonstra como o Replica Set do MongoDB garante **alta disponibilidade**, permitindo que o banco continue funcionando mesmo com a queda de um nó do cluster.

---

### 📄 Requisitos

- Docker instalado
- MongoDB Compass (opcional)
- Conhecimentos básicos de terminal
