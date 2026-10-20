<script setup>
import { onMounted, ref } from 'vue';
import { RouterLink, useRoute } from 'vue-router';

const route = useRoute();

const API_URL = 'http://localhost:3000';
const pet = ref({});
const tutor = ref({});
async function carregarPet() {
  try {
    const respostaPet = await fetch(`${API_URL}/pets/${route.params.id}`);
    if (!respostaPet.ok) {
      console.log('Opees, pet não encontrado!');
    }
    pet.value = await respostaPet.json();

    const respostaTutor = await fetch(
      `${API_URL}/tutores/${pet.value.tutorId}`,
    );
    tutor.value = respostaTutor.ok
      ? await respostaTutor.json()
      : { nome: 'Tutor não encontrado' };
  } catch (erro) {
    console.error('Erro ao carregar os dados do pet e do tutor:', erro);
  }
}
onMounted(carregarPet);
</script>

<template>
  <h1>Nome: {{ pet.nome }}</h1>
  <p>Espécie: {{ pet.especie }}</p>
</template>