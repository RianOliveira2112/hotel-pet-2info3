<script setup>
import { onMounted, ref } from 'vue';
import { RouterLink } from 'vue-router';
('onMounted');
const API_URL = 'http://localhost:3000';
('API_URL');

const pets = ref([]);
('pets');
const tutores = ref([]);
('tutores');

async function carregarDados() {
  const respostaPets = await fetch(`${API_URL}/pets`); //vai trazer a url da api do pets
  pets.value = await respostaPets.json(); // ele espera e depois chama em json

  const respostaTutores = await fetch(`${API_URL}/tutores`); //vai trazer a url da api dos tutores
  tutores.value = await respostaTutores.json(); // ele espera e depois chama em json
}
function nomeDoTutor(tutorId) {
  for (const tutor of tutores.value) {
    if (tutor.id === tutorId) {
      return tutor.nome;
    }
  }
} //função para excluir o pet 
 async function excluirPet(petId) {
  const confirmaExclusao = window.confirm('Tem certeza que deseja excluir este pet?');

  if (confirmaExclusao){
    await fetch(`${API_URL}/pets/${petId}`, {
      method: 'DELETE',
  });
    } await carregarDados(); // Recarrega a lista de pets após a exclusão
}

onMounted(() => {
  carregarDados();
});
</script>

<template>
  <div>
    <header class="mb-4">
      <h1 class="text-2xl font-bold">Listagem de Pets</h1>
      <p class="text-body-secondary mb-0">
        Listagem dos Pets cadastrados no sistema.
      </p>
    </header>
  </div>
  <table class="table table-striped table-hover">
    <thead>
      <tr>
        <th>ID</th>
        <th>Nome</th>
        <th>Espécie</th>
        <th>Tutor</th>
        <th>Ações</th>
      </tr>
   
    </thead>
    <tbody>
      <tr
        v-for="pet in pets"
        :key="pet.id"
      >
        <td>{{ pet.id }}</td>
        <td>{{ pet.nome }}</td>
        <td>{{ pet.especie }}</td>
        <td>{{ nomeDoTutor(pet.tutorId) }}</td>
        <td>
          <RouterLink :to="`/pets/${pet.id}`"> Visualizar </RouterLink> 
          
          <button class="btn btn-danger" @click="excluirPet(pet.id)"> Excluir </button>
        </td>
      </tr>
    </tbody>
  </table>
</template>

<script setup></script>