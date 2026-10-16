<script setup>
import { onMounted, ref } from 'vue';
import { RouterLink } from 'vue-router';
('onMounted');
const API_URL = 'http://localhost:3000';
('API_URL');

const tutores = ref([]);
('tutores');

async function carregarDados() {
  const respostaTutores = await fetch(`${API_URL}/tutores`); //vai trazer a url da api dos tutores
  tutores.value = await respostaTutores.json(); // ele espera e depois chama em json
}

 //função para excluir o pet 
 async function excluirTutor(tutorId) {
  const confirmaExclusao = window.confirm('Tem certeza que deseja excluir este tutor?');

  if (confirmaExclusao){
    await fetch(`${API_URL}/tutores/${tutorId}`, {
      method: 'DELETE',
    });
  }
  await carregarDados(); // Recarrega a lista de tutores após a exclusão
}

onMounted(carregarDados);
</script>

<template>
  <div>
    <header class="mb-4">
      <h1 class="text-2xl font-bold">Listagem de Tutores</h1>
      <p class="text-body-secondary mb-0">
        Listagem dos Tutores cadastrados no sistema.
      </p>
    </header>
  </div>
  <table class="table table-striped table-hover">
    <thead>
      <tr>
        <th>ID</th>
        <th>Nome</th>
        <th>Telefone</th>
        <th>Ações</th>
      </tr>
   
    </thead>
    <tbody>
      <tr
        v-for="tutor in tutores"
        :key="tutor.id"
      >
        <td>{{ tutor.id }}</td>
        <td>{{ tutor.nome }}</td>
        <td>{{ tutor.telefone }}</td>
        <td>
          <RouterLink :to="`/tutores/${tutor.id}`"> Visualizar </RouterLink> 
          
          <button class="btn btn-danger" @click="excluirTutor(tutor.id)"> Excluir </button>
        </td>
      </tr>
    </tbody>
  </table>
</template>

<script setup></script>