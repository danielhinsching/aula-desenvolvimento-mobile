---
title: 'Camada de serviço'
permalink: /loja-virtual-adicionar-produtos/camada-servico
---

# Camada de serviços

Vamos, inicialmente, criar as funções da camada de serviço, que fará a comunicação com a API do _backend_. Essa camada será posteriormente acessada pela camada de armazenamento, embora possa ser acessada diretamente dos componentes.

# Camada de serviço para categorias

Vamos criar uma camada de serviço para categorias. Essa camada será responsável por fazer a comunicação com a API do _backend_ para buscar e adicionar categorias. Para isso crie um arquivo chamado `src/services/category.js` e adicione o seguinte código:

```javascript
import axios from 'axios';

export default class CategoryService {
  async getCategories() {
    const response = await axios.get('/categories/');
    return response.data.results;
  }

  async createCategory(category) {
    const response = await axios.post('/categories/', category);
    return response.data;
  }
}
```

Note que criamos dois métodos: `getCategories` e `createCategory`. O método `getCategories` faz uma chamada GET para a URL `/categories/` e retorna as categorias cadastradas. O método `createCategory` faz uma chamada POST para a URL `/categories/` com a categoria a ser cadastrada.

# Camada de serviço para o uploader (envio de imagens)

Precisamos também de um serviço para o upload de imagens para a API. Vamos então criar o arquivo `src/services/uploader.js` com o seguinte conteúdo:

```javascript
import axios from 'axios';

export default class UploaderService {
  async uploadImage(image) {
    const formData = new FormData();
    formData.append('file', image);
    const response = await axios.post('/media/images/', formData, {
      headers: {
        'Content-Type': 'multipart/form-data',
      },
    });
    return response.data;
  }
}
```

Note que inicialmente criamos um objeto da classe formData para registar as informações da imagem. Ainda, adicionamos a imagem associada com o campo `file`, que é o nome do campo esperado pela API de backend. Em seguida, fizemos uma requisição do tipo POST para a API, informando no `header` que o `Content-Type` é `multipart/form-data`: este o tipo esperado pela API para envio de imagens, documentos, etc.

# Camada de serviço para produtos

Nós já temos um serviço para produtos, mas que ainda não tem implementada a criação de produtos. Vamos então adicionar essa funcionalidade. Para isso, edite o arquivo `src/services/product.js` e substitua o conteúdo pelo seguinte:

```javascript
import axios from 'axios';

export default class ProductService {
  async getProducts() {
    const response = await axios.get('/products/');
    return response.data.results;
  }

  async getProductByCategory(category_id) {
    const response = await axios.get(`/products/?category__id=${category_id}`);
    return response.data.results;
  }

  async createProduct(product) {
    const response = await axios.post('/products/', product);
    return response.data;
  }
}
```

Agora, temos o método `createProduct` que faz uma chamada POST para a URL `/products/` com o produto a ser cadastrado.

Na próxima etapa, vamos implementar a camada de armazenamento para acessar esses métodos.

<span style="display: flex; justify-content: space-between;"><span>[&lt; Tutorial da loja virtual - adicionar produtos](. 'Voltar')</span><span>[Criando a camada de armazenamento &gt;](camada-armazenamento.html 'Próximo')</span></span>
