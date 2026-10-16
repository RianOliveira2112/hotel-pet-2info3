<script setup>
  import { onMounted, ref } from 'vue';
  import {useRoute } from 'vue-router';

 // chamando a minha API para exibir os dados de tutor
  const API_URL = 'http://localhost:3000';
  const router = useRoute();

  // chamando a minha API para exibir os dados de tutor
  const tutores = ref([]);

  async function carregarTutores() {
    const resposta = await fetch(`${API_URL}/tutores`);
    console.log('tutores', resposta);
    // transformando os valores da minha API para o fromato JSON
    tutores.value = await resposta.json();
  }

  // chamando a minha API para salvar os dados de pet
  const novoTutor = ref({
    nome: '',
    telefone: '',
    tutorId: '',
  });

  async function salvarTutor() {
    // fazendo a requisição para a API para o servidor
    await fetch(`${API_URL}/tutores`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(novoTutor.value),
    });
    router.push({'name': 'tutores'});

    // limpando os campos do formulário
    novoTutor.value = {
      nome: '',
      telefone: '',
      tutorId: '',
    };
  }

  onMounted(carregarTutores);
</script>

<template>
  <div>
    <header class="mb-4">
      <h1 class="text-2xl font-bold">Listagem de Tutores</h1>
      <p class="text-body-secondary mb-0">Cadastro de Tutores no sistema.</p>
    </header>

    <!--
    <RouterLink
      class="btn btn-primary"
      :to="{ name: 'addTutor' }"
    >
      Adicionar Tutor
    </RouterLink>
    -->


    <form @submit.prevent="salvarTutor">
      <div class="col-md-6 mb-3">
        <label for="nome" class="form-label">Nome do tutor:</label>
        <input
          type="text"
          id="nome"
          v-model="novoTutor.nome"
          class="form-control"
          required
        />
      </div>


      <div class="col-md-6">
        <label for="telefone" class="form-label">Telefone:</label>
        <input
          type="text"
          id="telefone"
          v-model="novoTutor.telefone"
          class="form-control"
          required
        />
      </div>

      <button type="submit" class=" btn btn-success">Salvar Tutor</button>
    </form>
  </div>
</template>