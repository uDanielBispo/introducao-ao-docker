# Entendendo e Aplicando Docker

Este documento tem como objetivo apresentar os conceitos e a aplicação prática do Docker, com base no material didático disponibilizado pelo senhor Ayrton Teshima em seu canal no YouTube. Expressamos nossos mais sinceros agradecimentos pela valiosa contribuição.

Fonte: [https://www.youtube.com/watch?v=Kzcz-EVKBEQ](https://www.youtube.com/watch?v=Kzcz-EVKBEQ)

## O que é Docker?

Docker é uma plataforma de software que permite a criação e o gerenciamento de **contêineres virtuais**. Esses contêineres são ambientes isolados e portáteis, capazes de serem executados em qualquer sistema que possua o Docker instalado, seja uma máquina local ou um servidor.

A principal vantagem do Docker reside na capacidade de definir todas as configurações de um ambiente de aplicação uma única vez, gerando uma **imagem Docker**. Essa imagem pode ser distribuída e utilizada em diferentes sistemas/ambientes, garantindo consistência e reprodutibilidade.

Neste guia, utilizaremos imagens Docker pré-existentes, disponíveis no site oficial do Docker: [https://www.docker.com/](https://www.docker.com/)

## Problemática

Considere um cenário onde dois sistemas se comunicam: uma API desenvolvida em **Node.js** e uma página web em **PHP**. A API em Node.js, por sua vez, interage com um banco de dados **MySQL**. O desenvolvimento inicial ocorreu em um computador com sistema operacional Windows 10.

A complexidade surge quando o projeto precisa ser implantado em uma máquina diferente, exigindo a instalação e configuração de dependências em um sistema operacional distinto. Além disso, em equipes de desenvolvimento, cada membro pode utilizar um sistema operacional diferente ou versões distintas das mesmas dependências, gerando inconsistências e desafios de compatibilidade.

É nesse contexto que o Docker se torna uma solução eficaz, padronizando o ambiente de execução através de seus contêineres.

## Primeiros Passos: Configurando o Banco de Dados MySQL

### Criando a Imagem Docker do MySQL

Para iniciar, crie a estrutura de diretórios `/api/db` na raiz do seu projeto. Dentro do diretório `/api/db`, crie um arquivo chamado `Dockerfile` (sem extensão). Este arquivo definirá as características da sua imagem Docker, incluindo:

*   Sistema Operacional
*   Dependências
*   Arquivos a serem incluídos no contêiner
*   Comandos a serem executados

Para informações detalhadas sobre a sintaxe e as opções do `Dockerfile`, consulte a documentação oficial do Docker: [https://docs.docker.com/reference/dockerfile/](https://docs.docker.com/reference/dockerfile/)

O `Dockerfile` para o banco de dados MySQL conterá o seguinte:

```dockerfile
FROM mysql
ENV MYSQL_ROOT_PASSWORD sua_senha
```

Navegue até a pasta raiz do seu projeto no terminal e execute o seguinte comando para construir a imagem:

```bash
docker build -t mysql-image -f api/db/Dockerfile .
```

*   A flag `-t` (Tag) atribui um nome à imagem, neste caso, `mysql-image`.
*   A flag `-f` especifica o caminho para o arquivo `Dockerfile`.
*   O `.` no final indica que o contexto para a construção da imagem é o diretório atual (raiz do projeto).

Para listar todas as imagens Docker criadas, utilize o comando:

```bash
docker image ls
```

### Criando o Contêiner MySQL

Execute o comando abaixo para criar e iniciar o contêiner MySQL:

```bash
docker run -d --rm --name mysql-container mysql-image
```

| Trecho                   | Explicação                                                                                             |
| :----------------------- | :----------------------------------------------------------------------------------------------------- |
| `-d`                     | (`detached`) Executa o contêiner em segundo plano, sem bloquear o terminal.                            |
| `--rm`                   | Remove o contêiner automaticamente assim que ele for parado.                                           |
| `--name mysql-container` | Define o nome do contêiner como `mysql-container`, facilitando sua identificação e gerenciamento.      |
| `mysql-image`            | Nome da imagem Docker utilizada para criar o contêiner (deve ter sido construída previamente).         |

Para visualizar todos os contêineres em execução, utilize o comando:

```bash
docker ps
```

### Criando o Banco de Dados e Tabelas

Crie um arquivo chamado `script.sql` dentro do diretório `/api/db` e adicione o seguinte código SQL:

```sql
CREATE DATABASE IF NOT EXISTS 
    db_docker;
    
USE db_docker;

CREATE TABLE IF NOT EXISTS products (
    id INT(11) AUTO_INCREMENT,
    name VARCHAR(255),
    price DECIMAL(10,2),
    PRIMARY KEY (id)
);

INSERT INTO products VALUE(0, 'Curso Front-end especialista', 2500);
INSERT INTO products VALUE(0, 'Curso Fullstack', 3000);
```

Agora, execute este script SQL dentro do contêiner `mysql-container`:

```bash
docker exec -i mysql-container -uroot -psuasenha < api/db/script.sql
```

| Trecho                | Explicação                                                                                                                               |
| :-------------------- | :--------------------------------------------------------------------------------------------------------------------------------------- |
| `docker exec`         | Executa um comando **dentro de um contêiner já em execução**.                                                                            |
| `-i`                  | Mantém o STDIN aberto para receber dados (necessário para redirecionamento de entrada com `<`).                                          |
| `mysql-container`     | Nome do contêiner onde o comando será executado.                                                                                         |
| `mysql`               | Comando a ser executado **dentro do contêiner** (o cliente de linha de comando do MySQL).                                                |
| `-uroot`              | Realiza o login no MySQL como o usuário `root`.                                                                                          |
| `-psuasenha`          | Passa a senha do usuário `root` (o `-p` deve estar colado à senha, sem espaço).                                                          |
| `< api/db/script.sql` | **Redireciona um arquivo SQL do host para dentro do comando**, simulando a digitação do conteúdo diretamente no terminal do contêiner. |

**Observação:** O operador `<` funciona em ambientes **Linux/macOS** e em terminais como **Git Bash** ou **WSL**. No `CMD` padrão do Windows (e ocasionalmente no **PowerShell**), pode não funcionar corretamente.

Para usuários Windows, uma alternativa é utilizar `type` e `pipe` (`|`) no `CMD`:

```batch
type api\db\script.sql | docker exec -i mysql-container mysql -uroot -psuasenha
```

No `PowerShell`, você pode usar `${PWD}` no lugar de `$(pwd)`.

### Acessando o Contêiner MySQL

Para acessar o terminal do contêiner `mysql-container` e interagir diretamente com ele, utilize o seguinte comando:

```bash
docker exec -it mysql-container /bin/bash
```

| Trecho            | Significado                                                                                                 |
| :---------------- | :---------------------------------------------------------------------------------------------------------- |
| `docker exec`     | Executa um comando dentro de um contêiner em execução.                                                      |
| `-i`              | (`interactive`) Mantém o STDIN aberto, permitindo a digitação de comandos.                                  |
| `-t`              | (`tty`) Aloca um terminal para que você tenha uma sessão de shell com interface semelhante ao terminal local. |
| `mysql-container` | Nome (ou ID) do contêiner onde o comando será executado.                                                    |
| `/bin/bash`       | Comando a ser executado dentro do contêiner (neste caso, o shell Bash).                                     |

Após acessar o contêiner, você pode se conectar ao MySQL com `mysql -uroot -psuasenha`, selecionar o banco de dados com `USE db_docker;` e verificar os dados com `SELECT * FROM products;`.

### Persistência de Dados com Volumes

Ao sair do contêiner (utilizando `exit` duas vezes), você notará que todas as alterações (tabelas e bancos de dados) são perdidas. Para garantir a persistência dos dados, o Docker oferece o conceito de **Volumes**.

Volumes permitem compartilhar uma pasta do seu sistema host com o contêiner, criando um espelhamento. Dessa forma, as alterações realizadas dentro do contêiner são refletidas no host e vice-versa, garantindo que os dados não sejam perdidos mesmo após o contêiner ser interrompido ou removido.

Exemplo de execução de contêiner com volume:

```bash
docker run -d -v $(pwd)/api/db/data:/var/lib/mysql --rm --name mysql-container mysql-image
```

| Trecho                                | Significado                                                                                                                                |
| :------------------------------------ | :----------------------------------------------------------------------------------------------------------------------------------------- |
| `docker run`                          | Cria e inicia um novo contêiner a partir de uma imagem Docker.                                                                             |
| `-d`                                  | Executa o contêiner em modo *detached* (em segundo plano).                                                                                 |
| `-v $(pwd)/api/db/data:/var/lib/mysql` | Monta um volume: a pasta local `$(pwd)/api/db/data` será mapeada para `/var/lib/mysql` dentro do contêiner, onde o MySQL armazena seus dados. |
| `--rm`                                | Remove automaticamente o contêiner quando ele parar.                                                                                       |
| `--name mysql-container`              | Atribui o nome `mysql-container` ao contêiner, facilitando referências futuras.                                                            |
| `mysql-image`                         | Imagem Docker utilizada para criar o contêiner (deve já existir localmente).                                                              |

Novamente, para usuários Windows, o `CMD` ou `PowerShell` podem não reconhecer `$(pwd)`. Utilize a seguinte sintaxe:

```batch
docker run -d -v %cd%\api\db\data:/var/lib/mysql --rm --name mysql-container mysql-image
```

No `PowerShell`, você pode usar `${PWD}` no lugar de `$(pwd)`.

Após configurar o volume, execute novamente o script SQL para criar o banco de dados e as tabelas:

```bash
docker exec -i mysql-container -uroot -psuasenha < api/db/script.sql
```

## Criando o Contêiner Node.js

Para o contêiner Node.js, a estrutura do projeto deve ser a seguinte:

```
meu-projeto/
├── api/
│   └── db/
│       └── Dockerfile
│       └── script.sql
├── src/
│   └── index.js
├── package-lock.json
├── package.json
```

Certifique-se de ter o Node.js instalado em sua máquina. Em seguida, inicialize o projeto Node.js e instale as dependências:

1.  Inicialize o projeto Node.js:
    ```bash
    npm init
    ```
    Siga as instruções para configurar o `package.json`.

2.  Instale o `nodemon` (para desenvolvimento, para reiniciar o servidor automaticamente):
    ```bash
    npm install --save-dev nodemon
    ```

3.  Instale `express` (para rotas) e `mysql2` (para conexão com MySQL):
    ```bash
    npm install --save express mysql2
    ```

No seu arquivo `package.json`, adicione ou modifique a seção `scripts` para incluir o comando `start`:

```json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "nodemon src/index.js"
}
```

Crie a pasta `src` e, dentro dela, o arquivo `index.js` com o seguinte conteúdo:

```javascript
const express = require("express")
const mysql = require("mysql2")

const app = express()

const connection = mysql.createConnection({
    host: 'ip-do-seu-container',
    user: 'root',
    password: 'suasenha',
    database: 'db_docker'
})

connection.connect();

app.get('/products', function(req, res) {
    connection.query('SELECT * FROM products', function (error, results) {
        if(error) throw error

        res.send(results.map(item=> ({name: item.name, price: item.price})))
    })
})


app.listen(9001, '0.0.0.0', function(){
    console.log('Listening to http://localhost:9001');
})
```

Para descobrir o IP do seu contêiner MySQL, execute no terminal:

```bash
docker inspect mysql-container
```

Procure por `


"IPAddress" e cole o valor encontrado no parâmetro `host` do seu `index.js`.

### Criando o Dockerfile do Node.js

Crie um arquivo `Dockerfile` na pasta `/api`, de forma que a estrutura do projeto fique assim:

```
meu-projeto/
├── api/
│   └── db/
│       └── Dockerfile
│       └── script.sql
│   └── Dockerfile  <-- Novo Dockerfile para o Node.js
├── src/
│   └── index.js
├── package-lock.json
├── package.json
```

No `Dockerfile` recém-criado, insira o seguinte conteúdo:

```dockerfile
FROM node:10-slim
WORKDIR /home/node/app
CMD npm start
```

O comando `CMD npm start` será executado quando o contêiner for iniciado, iniciando o servidor Node.js.

Execute o comando abaixo na pasta raiz do projeto para construir a imagem Node.js:

```bash
docker build -t node-image -f api/Dockerfile .
```

### Rodando o Contêiner Node.js

Para criar e iniciar o contêiner Node.js, utilize o seguinte comando:

```bash
docker run -d -v $(pwd)/api:/home/node/app -p 9001:9001 --rm --name node-container node-image
```

| Trecho                        | Explicação                                                                                                                               |
| :---------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------- |
| `docker run`                  | Cria e inicia um novo contêiner.                                                                                                         |
| `-d`                          | Executa o contêiner em segundo plano (modo *detached*).                                                                                  |
| `-v $(pwd)/api:home/node/app` | Monta um volume local (bind mount): a pasta `api/` do host será montada em `home/node/app` dentro do contêiner, onde a aplicação Node.js estará. |
| `-p 9001:9001`                | Mapeia a porta 9001 do host para a porta 9001 do contêiner, permitindo o acesso à aplicação Node.js.                                     |
| `--rm`                        | Remove o contêiner automaticamente quando ele parar.                                                                                     |
| `--name node-container`       | Define o nome do contêiner como `node-container`.                                                                                        |
| `node-image`                  | Nome da imagem Docker que será usada.                                                                                                    |

Após a execução bem-sucedida, você poderá acessar os dados da API Node.js através do seu navegador, visitando `http://localhost:9001/products`.

## Criando o Contêiner PHP

Para o contêiner PHP, crie uma pasta chamada `website` na raiz do projeto. Dentro dela, inclua um arquivo `index.php` e outro `Dockerfile`. A estrutura do projeto ficará assim:

```
meu-projeto/
├── api/
│   └── db/
│       └── Dockerfile
│       └── script.sql
├── src/
│   └── index.js
├── Dockerfile
├── package-lock.json
├── package.json
├── website/
│   └── Dockerfile  <-- Novo Dockerfile para o PHP
│   └── index.php
```

No arquivo `index.php`, insira o seguinte código PHP:

```php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <?php 
        $result = file_get_contents("http://IPAdress-do-seu-container:9001/products");
        $products = json_decode($result);
    ?>

    <table>
        <thead>
            <tr>
                <th>Produto</th>
                <th>Preço</th>
            </tr>
        </thead>
        <tbody>
            <?php foreach($products as $product):  ?>
                <tr>
                    <td><?php echo $product->name; ?></td>
                    <td><?php echo $product->price; ?></td>
                </tr> 
            <?php endforeach;  ?>
        </tbody>
    </table>
</body>
</html>
```

Este código PHP permitirá que a página web consuma os dados da API Node.js.

No `Dockerfile` localizado em `website/`, inclua:

```dockerfile
FROM php:7.2-apache
WORKDIR /var/www/html
```

Construa a imagem PHP com o seguinte comando:

```bash
docker build -t php-image -f website/Dockerfile .
```

Crie o contêiner PHP a partir desta imagem:

```bash
docker run -d -v $(pwd)/website:/var/www/html -p 8888:80 --rm --name php-container php-image
```

| Trecho                 | Significado                                                                                                                                |
| :--------------------- | :----------------------------------------------------------------------------------------------------------------------------------------- |
| `-d`                   | Executa o contêiner em segundo plano (detached).                                                                                           |
| `-v ...:/var/www/html` | Monta a pasta local `website/` dentro do contêiner no caminho padrão do Apache para arquivos da web, garantindo que o `index.php` seja acessível. |
| `-p 8888:80`           | Expõe a porta 80 do contêiner (porta padrão do Apache) na porta 8888 do host, permitindo o acesso via `http://localhost:8888`.               |
| `--rm`                 | Remove o contêiner automaticamente ao ser parado.                                                                                          |
| `--name php-container` | Atribui o nome `php-container` ao contêiner.                                                                                               |
| `php-image`            | Nome da imagem Docker a ser usada.                                                                                                         |

Finalmente, você poderá acessar a aplicação PHP através do seu navegador, visitando `http://localhost:8888` para visualizar os dados.

## Conclusão

Neste guia, demonstramos a utilização do Docker para orquestrar um ambiente de aplicação completo, composto por um banco de dados MySQL, uma API em Node.js e um frontend em PHP. Através da conteinerização, foi possível garantir a consistência do ambiente de desenvolvimento e produção, facilitando a implantação e o gerenciamento de aplicações complexas.

