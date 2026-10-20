<script setup>
import { onMounted, ref } from 'vue';
import { RouterLink } from 'vue-router';

const API_URL = 'http://localhost:3000';

const tutores = ref([]);
const pets = ref([]);

async function carregarDados() {
  const respostaTutores = await fetch(`${API_URL}/tutores`);
  tutores.value = await respostaTutores.json();

  const respostaPets = await fetch(`${API_URL}/pets`);
  pets.value = await respostaPets.json();
}

function petsDoTutor(tutorId) {
  return pets.value.filter((pet) => String(pet.tutorId) === String(tutorId));
}

function quantidadePets(tutorId) {
  return petsDoTutor(tutorId).length;
}

function nomesPets(tutorId) {
  return petsDoTutor(tutorId)
    .map((pet) => pet.nome)
    .join(', ');
}

async function excluirTutor(tutorId) {
  const tutorComPets = petsDoTutor(tutorId);

  if (tutorComPets.length > 0) {
    alert(
      'Este tutor possui pets vinculados. Remova ou transfira os pets antes de excluir o tutor.',
    );
    return;
  }

  const confirmar = window.confirm('Deseja realmente remover este tutor?');

  if (!confirmar) {
    return;
  }

  await fetch(`${API_URL}/tutores/${tutorId}`, {
    method: 'DELETE',
  });

  await carregarDados();
}

onMounted(() => {
  carregarDados();
});
</script>

<template>
  <div>
    <header class="mb-4 d-flex justify-content-between align-items-center">
      <div>
        <h1 class="text-2xl font-bold">Listagem de tutores</h1>
        <p class="text-body-secondary mb-0">Tutores cadastrados no sistema.</p>
      </div>

      <RouterLink
        class="btn btn-success"
        to="/tutores/novo"
      >
        Novo tutor
      </RouterLink>
    </header>

    <table class="table table-striped table-hover">
      <thead>
        <tr>
          <th>ID</th>
          <th>Nome completo</th>
          <th>Telefone</th>
          <th>E-mail</th>
          <th>CPF/CNPJ</th>
          <th>Quantidade de pets</th>
          <th>Pets vinculados</th>
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
          <td>{{ tutor.email || 'Não informado' }}</td>
          <td>{{ tutor.cpfCnpj || 'Não informado' }}</td>
          <td>{{ quantidadePets(tutor.id) }}</td>
          <td>{{ nomesPets(tutor.id) || 'Nenhum pet' }}</td>
          <td>
            <RouterLink
              class="btn btn-sm btn-primary me-2"
              :to="`/tutores/${tutor.id}`"
            >
              Ver
            </RouterLink>
            <button
              class="btn btn-sm btn-danger"
              @click="excluirTutor(tutor.id)"
            >
              Excluir
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>