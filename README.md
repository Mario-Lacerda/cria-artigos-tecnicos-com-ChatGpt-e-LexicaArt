# Desafio Dio - Criando Artigos Técnicos com ChatGPT e Lexica.art



**projeto ainda mais completo com ainda mais códigos para criar artigos técnicos com ChatGPT e Lexica.art:**

**Introdução**

Neste projeto é para construir uma aplicação full-stack que usa o ChatGPT, o Lexica.art e o Google Cloud Storage para criar e publicar artigos técnicos. A aplicação será construída usando Node.js, React, MongoDB, Socket.IO e a API do Google Cloud Storage.



### **Pré-requisitos**

- Conta da OpenAI

- Conta da Lexica.art

- Conta do Google Cloud Storage

- Node.js 14 ou superior

- npm 6 ou superior

- MongoDB

  

### **Configuração**

### **Servidor**



1. #### Crie um novo projeto Node.js:

   plaintext

   

   ```plaintext
   npm init -y
   ```

   

2. #### Instale as seguintes dependências:

   plaintext

   

   ```plaintext
   npm install express body-parser mongoose fetch multer socket.io
   ```

   

3. #### Crie um arquivo `server.js` com o seguinte código:

   javascript

   

   ```javascript
   const express = require('express')
   const bodyParser = require('body-parser')
   const mongoose = require('mongoose')
   const fetch = require('node-fetch')
   const multer = require('multer')
   const socketIO = require('socket.io')
   
   const app = express()
   
   app.use(bodyParser.json())
   
   mongoose.connect('mongodb://localhost:27017/articles', {
     useNewUrlParser: true,
     useUnifiedTopology: true
   })
   
   const ArticleSchema = new mongoose.Schema({
     title: String,
     body: String,
     author: String,
     image: String
   })
   
   const Article = mongoose.model('Article', ArticleSchema)
   
   const storage = multer.diskStorage({
     destination: './uploads/',
     filename: (req, file, cb) => {
       cb(null, file.originalname)
     }
   })
   
   const upload = multer({ storage })
   
   app.post('/article', upload.single('image'), async (req, res) => {
     const { prompt, author } = req.body
   
     const response = await fetch('https://generativelanguage.googleapis.com/v1beta2/models/chat-bison-001:generateMessage?key={{API_KEY}}', {
       method: 'POST',
       headers: {
         'Content-Type': 'application/json'
       },
       body: JSON.stringify({
         input: {
           text: prompt
         }
       })
     })
   
     const data = await response.json()
   
     const article = {
       title: data.candidates[0].content,
       body: data.candidates[1].content,
       author: author,
       image: req.file.originalname
     }
   
     const lexicoResponse = await fetch('https://api.lexica.art/v1/articles', {
       method: 'POST',
       headers: {
         'Content-Type': 'application/json',
         'Authorization': `Bearer {{API_KEY}}`
       },
       body: JSON.stringify(article)
     })
   
     const lexicoData = await lexicoResponse.json()
   
     const newArticle = new Article({
       title: lexicoData.article.title,
       body: lexicoData.article.body,
       author: author,
       image: lexicoData.article.image
     })
   
     newArticle.save()
   
     res.json(lexicoData)
   })
   
   app.get('/articles', async (req, res) => {
     const articles = await Article.find()
   
     res.json(articles)
   })
   
   const io = socketIO(server)
   
   io.on('connection', (socket) => {
     console.log('Novo usuário conectado')
   
     socket.on('new article', (article) => {
       io.emit('new article', article)
     })
   
     socket.on('disconnect', () => {
       console.log('Usuário desconectado')
     })
   })
   
   app.listen(3000, () => {
     console.log('Servidor rodando na porta 3000')
   })
   ```



### **Cliente**



1. #### Crie um novo projeto React:

   plaintext

   

   ```plaintext
   npx create-react-app articles
   ```

   

2. #### No diretório do projeto React, instale as dependências `axios`, `socket.io-client` e `react-dropzone`:

   plaintext

   

   ```plaintext
   npm install axios socket.io-client react-dropzone
   ```

   

3. #### No arquivo `App.js`, substitua o código existente pelo seguinte:

   javascript

   

   ```javascript
   import React, { useState, useEffect } from 'react'
   import axios from 'axios'
   import socketIOClient from 'socket.io-client'
   import Dropzone from 'react-dropzone'
   
   const App = () => {
     const [prompt, setPrompt] = useState('')
     const [author, setAuthor] = useState('')
     const [articles, setArticles] = useState([])
     const [image, setImage] = useState(null)
   
     useEffect(() => {
       const socket = socketIOClient('http://localhost:3000')
   
       socket.on('new article', (article) => {
         setArticles([article, ...articles])
       })
   
       return () => {
         socket.disconnect()
       }
     }, [articles])
   
     const handleSubmit = async (e) => {
       e.preventDefault()
   
       const formData = new FormData()
       formData.append('prompt', prompt)
       formData.append('author', author)
       formData.append('image', image)
   
       const res = await axios.post('http://localhost:3000/article', formData)
   
       socket.emit('new article', res.data.article)
     }
   
     const onDrop = (acceptedFiles) => {
       setImage(acceptedFiles[0])
     }
   
     return (
       <div>
         <h1>Criador de Artigos Técnicos</h1>
         <form onSubmit={handleSubmit}>
           <label htmlFor="prompt">Pergunta:</label>
           <input type="text" id="prompt" value={prompt} onChange={(e) => setPrompt(e.target.value)} />
           <label htmlFor="author">Autor:</label>
           <input type="text" id="author" value={author} onChange={(e) => setAuthor(e.target.value)} />
           <Dropzone onDrop={onDrop} multiple={false}>
             {({ getRootProps, getInputProps }) => (
               <div {...getRootProps()}>
                 <input {...getInputProps()} />
                 <p>Arraste e solte uma imagem ou clique aqui para selecionar</p>
               </div>
             )}
           </Dropzone>
           <button type="submit">Enviar</button>
         </form>
         <div>
           {articles.map((article) => (
             <div key={article._id}>
               <h2>{article.title}</h2>
               <p>{article.body}</p>
               <p>Por {article.author}</p>
               <img src={article.image} alt={article.title} />
             </div>
           ))}
         </div>
       </div>
     )
   }
   
   export default App
   ```

   

4. #### Rode o servidor Node.js:

   plaintext

   

   ```plaintext
   node server.js
   ```

   

5. #### Rode a aplicação React:

   plaintext

   

   ```plaintext
   cd articles
   npm start
   ```



### **Uso**

Abra o navegador e vá para `http://localhost:3000`. Você verá um formulário onde poderá digitar uma pergunta para o ChatGPT, o nome do autor do artigo e selecionar uma imagem. Clique no botão "Enviar" para enviar sua pergunta. O ChatGPT irá gerar um artigo técnico, publicá-lo na página e enviar uma notificação em tempo real para todos os usuários conectados.



## **Conclusão**



Este é um projeto full-stack ainda mais completo que demonstra como usar o ChatGPT, o Lexica.art e o Google Cloud Storage para criar e publicar artigos técnicos com imagens em tempo real. Este projeto é mais abrangente do que os anteriores porque usa o React Dropzone para upload de imagens e o Google Cloud Storage para armazenar as imagens. Você pode expandir este projeto adicionando recursos como autenticação do usuário, comentários e muito mais.
