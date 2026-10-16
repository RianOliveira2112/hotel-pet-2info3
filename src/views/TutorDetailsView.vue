<script setup>
import { onMounted, ref} from 'vue';

import {RouterLink, useRoute} from 'vue-router';

const API_URL = 'http://localhost:3000';


const tutor = ref({});

const route = useRoute();


async function carregarTutor() {
  try{
    const respotasTutor = await(fetch(`${API_URL}/tutores/${route.params.id}`));    
    tutor.value = await respotasTutor.json();
    console.log('tutor', tutor.value);

    const respostaTutor = await fetch(`${API_URL}/tutores/${tutor.value.tutorId}`);

    tutor.value =  respostaTutor.ok ? await respostaTutor.json() : {nome: 'Tutor não encontrado'};
    
} catch (error) {
        console.error('Erro ao carregar os dados do tutor:', error);
    }
}

onMounted(carregarTutor);


</script>


<template>
    <h1>Nome: {{ tutor.nome }}</h1>
    <p>Telefone: {{ tutor.telefone }}</p>

</template>