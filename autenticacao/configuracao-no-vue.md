---
title: 'Configuração no Vue'
description: 'Configuração geral no Vue e integração com backend'
permalink: /autenticacao/configuracao-no-vue
---

# Instalando as dependências do Passage

Para instalar as dependências do Passage, execute o seguinte comando:

```bash
npm install @passageidentity/passage-elements @passageidentity/passage-auth
```

# Criando a página de Login

Vamos inicialmente criar a página de login. Para isso, vamos editar o arquivo `src/views/LoginView.vue` e adicionar o seguinte código:

```vue
<script setup>
import '@passageidentity/passage-elements/passage-auth';
</script>

<template>
  <h1>Login</h1>
  <div class="authContainer">
    <passage-auth
      app-id="APP_ID_QUE_ESTÁ_DISPONÍVEL_NO_DASHBOARD_DO_PASSAGE"
    ></passage-auth>
  </div>
</template>

<style></style>
```

Note que deve ser substituído o valor `APP_ID_QUE_ESTÁ_DISPONÍVEL_NO_DASHBOARD_DO_PASSAGE` pelo valor do `Application ID` que está disponível no dashboard do Passage.

# Configurando o pinia e api para registro da autenticação

Vamos criar um arquivo para a chamada de API. Para isso, crie um arquivo chamado `src/services/auth.js` e adicione o seguinte código:

```javascript
import axios from 'axios';

export default class AuthService {
  async postUserToken(token) {
    const response = await axios.get('/users/me/', {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });
    return response.data;
  }
}
```

Note que criamos um método `postUserToken` que faz uma chamada POST para a URL `/users/me/` com o _token_ do usuário. Esta URL está configurada no _backend_, e é responsável para receber o _token_ enviado pelo Passage, validar o _token_ e retornar o usuário autenticado. Caso o usuário ainda não esteja criado no _backend_, ele será criado e autenticado.

Em seguida, vamos criar um _store_ para armazenar o _token_ do usuário. Para isso, crie um arquivo chamado `src/stores/auth.js` e adicione o seguinte código:

```javascript
import { ref } from 'vue';
import { defineStore } from 'pinia';

import AuthService from '@/services/auth';
const authService = new AuthService();

export const useAuthStore = defineStore('auth', () => {
  const user = ref({});

  async function setToken(token) {
    user.value = await authService.postUserToken(token);
  }

  function unsetToken() {
    user.value = {};
  }

  return { user, setToken, unsetToken };
});
```

Neste arquivo, criamos um _store_ chamado `auth` que possui um _ref_ chamado `user` e dois métodos: `setToken` e `unsetToken`. O método `setToken` faz uma chamada para o método `postUserToken` do `AuthService` e armazena o usuário autenticado no _ref_ `user`. O método `unsetToken` limpa o _ref_ `user`.

# Ajustes no home para integração com o backend

O Passage foi configurado, na etapa anterior para que a URL de redirecionamento seja `/`. Para que o login funcione corretamente, é necessário fazer alguns ajustes ao redirecionar o usuário da página de login. Note que no nosso exemplo, a URL `/` é a página inicial do PWA. Para isso, vamos editar o arquivo `src/views/HomeView.vue` e adicionar o seguinte código:

```vue
<script setup>
import ProductList from '@/components/ProductList.vue';

import { onMounted } from 'vue';
import { PassageUser } from '@passageidentity/passage-elements/passage-user';
import { useAuthStore } from '@/stores/auth';

const authStore = useAuthStore();

const getUserInfo = async () => {
  try {
    const authToken = localStorage.getItem('psg_auth_token');
    const passageUser = new PassageUser(authToken);
    const user = await passageUser.userInfo(authToken);
    if (user) {
      await authStore.setToken(authToken);
    } else {
      authStore.unsetToken();
    }
  } catch (error) {
    authStore.unsetToken();
  }
};

onMounted(() => {
  getUserInfo();
});
</script>

<template>
  <product-list />
</template>
```

Note que ao carregar a página, o método `getUserInfo` é chamado. Este método faz uma chamada para o método `userInfo` do `PassageUser` e verifica se o usuário está autenticado. Caso o usuário esteja autenticado, o token é armazenado no _store_ `auth`. Caso o usuário não esteja autenticado, o token é removido do _store_ `auth`.

# Verificando a autenticação

Para verificar se o usuário está autenticado, você pode abrir o Vue DevTools e verificar o _store_ `auth`. Caso o usuário esteja autenticado, o _store_ `auth` deve conter as informações do usuário.

# Enviando o token nas requisições para o backend

Como última etapa, vamos garantir que o token seja enviado nas requisições para o _backend_. Para isso, vamos editar o arquivo `src/plugins/axios.js` e adicionar o seguinte código:

```javascript
axios.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('psg_auth_token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  },
);
```

Como isso, o token será enviado em todas as requisições para o _backend_, como um cabeçalho `Authorization`.

<span style="display: flex; justify-content: space-between;"><span>[&lt; Configuração do Passage](configuracao-passage.html 'Voltar')</span> <span>[Sumário &gt;](../ 'Próximo')</span></span>
