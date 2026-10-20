<script setup>
import { onMounted, ref } from 'vue';
import { useRoute, useRouter } from 'vue-router';

const route = useRoute();
const router = useRouter();

const API_URL = 'http://localhost:3000';
const tutor = ref({});
const pets = ref([]);
const novoPet = ref({
  nome: '',
  especie: '',
  tutorId: route.params.id,
});

async function carregarDados() {
  try {
    const respostaTutor = await fetch(`${API_URL}/tutores/${route.params.id}`);
    tutor.value = await respostaTutor.json();

    const respostaPets = await fetch(
      `${API_URL}/pets?tutorId=${route.params.id}`,
    );
    pets.value = await respostaPets.json();
  } catch (erro) {
    console.error('Erro ao carregar os dados do tutor e dos pets:', erro);
  }
}

async function salvarPet() {
  if (!novoPet.value.nome.trim() || !novoPet.value.especie.trim()) {
    alert('Preencha o nome e a espécie do pet antes de salvar.');
    return;
  }

  const dadosParaEnviar = {
    nome: novoPet.value.nome.trim(),
    especie: novoPet.value.especie,
    tutorId: String(route.params.id),
  };

  await fetch(`${API_URL}/pets`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(dadosParaEnviar),
  });

  novoPet.value = {
    nome: '',
    especie: '',
    tutorId: route.params.id,
  };

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
        <h1 class="text-2xl font-bold">Detalhes do tutor</h1>
        <p class="text-body-secondary mb-0">
          Informações do tutor e seus pets.
        </p>
      </div>

      <button
        class="btn btn-secondary"
        @click="router.push({ name: 'tutores' })"
      >
        Voltar
      </button>
    </header>

    <section class="mb-5 border rounded p-3">
      <h2 class="h4">Dados do tutor</h2>
      <p class="mb-1"><strong>ID:</strong> {{ tutor.id }}</p>
      <p class="mb-1"><strong>Nome:</strong> {{ tutor.nome }}</p>
      <p class="mb-1"><strong>Telefone:</strong> {{ tutor.telefone }}</p>
      <p class="mb-1">
        <strong>E-mail:</strong> {{ tutor.email || 'Não informado' }}
      </p>
      <p class="mb-1">
        <strong>CPF/CNPJ:</strong> {{ tutor.cpfCnpj || 'Não informado' }}
      </p>
      <p class="mb-1">
        <strong>CEP:</strong> {{ tutor.endereco?.cep || 'Não informado' }}
      </p>
      <p class="mb-1">
        <strong>Rua:</strong> {{ tutor.endereco?.rua || 'Não informado' }}
      </p>
      <p class="mb-1">
        <strong>Número:</strong> {{ tutor.endereco?.numero || 'Não informado' }}
      </p>
      <p class="mb-1">
        <strong>Bairro:</strong> {{ tutor.endereco?.bairro || 'Não informado' }}
      </p>
      <p class="mb-1">
        <strong>Cidade:</strong> {{ tutor.endereco?.cidade || 'Não informado' }}
      </p>
      <p class="mb-0">
        <strong>Estado:</strong> {{ tutor.endereco?.estado || 'Não informado' }}
      </p>
    </section>

    <section class="mb-5">
      <h2 class="h4">Pets vinculados</h2>

      <div
        v-if="pets.length === 0"
        class="alert alert-light border"
      >
        Nenhum pet vinculado a este tutor.
      </div>

      <table
        v-else
        class="table table-striped table-hover"
      >
        <thead>
          <tr>
            <th>ID</th>
            <th>Nome</th>
            <th>Espécie</th>
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
            <td>
              <RouterLink
                class="btn btn-sm btn-primary"
                :to="`/pets/${pet.id}`"
              >
                Ver
              </RouterLink>
            </td>
          </tr>
        </tbody>
      </table>
    </section>

    <section class="border rounded p-3">
      <h2 class="h4">Cadastrar pet para esse tutor</h2>

      <form @submit.prevent="salvarPet">
        <div class="row g-3">
          <div class="col-md-5">
            <label
              for="petNome"
              class="form-label"
              >Nome do pet:</label
            >
            <input
              id="petNome"
              v-model="novoPet.nome"
              type="text"
              class="form-control"
            />
          </div>

          <div class="col-md-5">
            <label
              for="petEspecie"
              class="form-label"
              >Espécie:</label
            >
            <select
              id="petEspecie"
              v-model="novoPet.especie"
              class="form-select"
            >
              <option value="">Selecione</option>
              <option value="Cachorro">Cachorro</option>
              <option value="Gato">Gato</option>
            </select>
          </div>

          <div class="col-md-2 d-flex align-items-end">
            <button
              type="submit"
              class="btn btn-success w-100"
            >
              Salvar
            </button>
          </div>
        </div>
      </form>
    </section>
  </div>
</template>