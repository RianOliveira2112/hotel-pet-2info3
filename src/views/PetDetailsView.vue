<script setup>
import { onMounted, ref } from 'vue';
import {RouterLink, useRoute} from 'vue-router';

//chamando a minha UseRoute
const route = useRoute();


// chamando nossa API
const API_URL = 'http://localhost:3000/pets';


// criando variaveis reativas para armazenar os dados do pet e do tutor
const pet = ref({});
const tutor = ref({});

// função para carregar os dados do pet e do tutor
async function carregarPet() {
try {
    const respostaPet = await fetch(`${API_URL}/pets/${route.params.id}`);
    if (!respostaPet.ok) {
        console.log('Opees, pet não encontrado');
    }
   pet.value = await respostaPet.json();

    const respostaTutor = await fetch(`${API_URL}/tutores/${pet.value.tutorId}`);
  tutor.value = await respostaTutor.ok ? await respostaTutor.json() : { nome: 'Não Especificado' };
  } catch (error) {
    console.error('Erro ao carregar os dados do pet e do tutor:', error);
  }
}

onMounted(() => {carregarPet()});
</script>


<template>
    <h1>Nome: {{ pet.nome }}</h1>
    <p>Espécie: {{ pet.especie }}</p>
    <p>Tutor: {{ nomeDoTutor(pet.tutorId) }}</p>
</template>