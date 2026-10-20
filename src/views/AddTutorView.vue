<script setup>
import { computed, ref } from 'vue';
import { useRouter } from 'vue-router';

const API_URL = 'http://localhost:3000';
const router = useRouter();

const novoTutor = ref({
  nome: '',
  email: '',
  telefone: '',
  cpfCnpj: '',
  endereco: {
    rua: '',
    numero: '',
    bairro: '',
    cidade: '',
    estado: '',
    cep: '',
  },
});

const tentouEnviar = ref(false);

function aplicarMascaraTelefone(valor) {
  const apenasNumeros = valor.replace(/\D/g, '').slice(0, 11);

  if (apenasNumeros.length <= 10) {
    return apenasNumeros.replace(
      /(\d{2})(\d{0,4})(\d{0,4})/,
      (match, a, b, c) => {
        if (b && c) return `(${a}) ${b}-${c}`;
        if (b) return `(${a}) ${b}`;
        if (a) return `(${a}`;
        return '';
      },
    );
  }

  return apenasNumeros.replace(/(\d{2})(\d{5})(\d{0,4})/, (match, a, b, c) => {
    if (c) return `(${a}) ${b}-${c}`;
    if (b) return `(${a}) ${b}`;
    return `(${a}`;
  });
}

function aplicarMascaraCpfCnpj(valor) {
  const numeros = valor.replace(/\D/g, '').slice(0, 14);

  if (numeros.length <= 11) {
    return numeros
      .replace(/(\d{3})(\d)/, '$1.$2')
      .replace(/(\d{3})(\d)/, '$1.$2')
      .replace(/(\d{3})(\d{1,2})$/, '$1-$2');
  }

  return numeros
    .replace(/(\d{2})(\d)/, '$1.$2')
    .replace(/(\d{3})(\d)/, '$1.$2')
    .replace(/(\d{3})(\d)/, '$1/$2')
    .replace(/(\d{4})(\d{1,2})$/, '$1-$2');
}

function formatarEmail(valor) {
  return valor.trim().toLowerCase();
}

function aplicarMascaraCep(valor) {
  return valor
    .replace(/\D/g, '')
    .slice(0, 8)
    .replace(/(\d{5})(\d)/, '$1-$2');
}

async function buscarCep() {
  const cep = novoTutor.value.endereco.cep.replace(/\D/g, '');

  if (cep.length !== 8) {
    return;
  }

  try {
    const resposta = await fetch(`https://viacep.com.br/ws/${cep}/json/`);
    const dados = await resposta.json();

    if (dados.erro) {
      alert('CEP não encontrado.');
      return;
    }

    novoTutor.value.endereco.rua = dados.logradouro || '';
    novoTutor.value.endereco.bairro = dados.bairro || '';
    novoTutor.value.endereco.cidade = dados.localidade || '';
    novoTutor.value.endereco.estado = dados.uf || '';
  } catch (erro) {
    console.error('Erro ao buscar CEP:', erro);
    alert('Não foi possível buscar o CEP.');
  }
}

const nomeValido = computed(() => novoTutor.value.nome.trim().length > 0);
const telefoneValido = computed(
  () => novoTutor.value.telefone.replace(/\D/g, '').length >= 10,
);
const cpfValido = computed(() => {
  const cpfNumeros = novoTutor.value.cpfCnpj.replace(/\D/g, '');
  return cpfNumeros.length === 0 || cpfNumeros.length >= 11;
});
const formValido = computed(
  () => nomeValido.value && telefoneValido.value && cpfValido.value,
);

function atualizarTelefone(event) {
  novoTutor.value.telefone = aplicarMascaraTelefone(event.target.value);
}

function atualizarCpfCnpj(event) {
  novoTutor.value.cpfCnpj = aplicarMascaraCpfCnpj(event.target.value);
}

function validarAntesDeSalvar() {
  tentouEnviar.value = true;

  if (!formValido.value) {
    return false;
  }

  return true;
}

async function salvarTutor() {
  if (!validarAntesDeSalvar()) {
    return;
  }

  const dadosParaEnviar = {
    nome: novoTutor.value.nome.trim(),
    telefone: novoTutor.value.telefone.replace(/\D/g, ''),
    endereco: {
      rua: novoTutor.value.endereco.rua,
      numero: novoTutor.value.endereco.numero,
      bairro: novoTutor.value.endereco.bairro,
      cidade: novoTutor.value.endereco.cidade,
      estado: novoTutor.value.endereco.estado,
      cep: novoTutor.value.endereco.cep.replace(/\D/g, ''),
    },
  };

  if (novoTutor.value.email.trim()) {
    dadosParaEnviar.email = formatarEmail(novoTutor.value.email);
  }

  if (novoTutor.value.cpfCnpj.trim()) {
    dadosParaEnviar.cpfCnpj = novoTutor.value.cpfCnpj.replace(/\D/g, '');
  }

  await fetch(`${API_URL}/tutores`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(dadosParaEnviar),
  });

  router.push({ name: 'tutores' });
}
</script>

<template>
  <div>
    <header class="mb-4">
      <h1 class="text-2xl font-bold">Cadastro de tutor</h1>
      <p class="text-body-secondary mb-0">Preencha os dados do tutor.</p>
    </header>

    <form @submit.prevent="salvarTutor">
      <div class="col-md-6 mb-3">
        <label
          for="nome"
          class="form-label"
          >Nome completo:</label
        >
        <input
          id="nome"
          v-model="novoTutor.nome"
          type="text"
          class="form-control"
        />
        <small
          v-if="tentouEnviar && !nomeValido"
          class="text-danger d-block"
        >
          campo obrigatorio, preencha-o antes de continuar
        </small>
      </div>

      <div class="col-md-6 mb-3">
        <label
          for="telefone"
          class="form-label"
          >Telefone:</label
        >
        <input
          id="telefone"
          :value="novoTutor.telefone"
          type="text"
          class="form-control"
          @input="atualizarTelefone"
        />
        <small
          v-if="tentouEnviar && !telefoneValido"
          class="text-danger d-block"
        >
          campo obrigatorio, preencha-o antes de continuar
        </small>
      </div>

      <div class="col-md-6 mb-3">
        <label
          for="email"
          class="form-label"
          >E-mail:</label
        >
        <input
          id="email"
          v-model="novoTutor.email"
          type="email"
          class="form-control"
          @blur="novoTutor.email = formatarEmail(novoTutor.email)"
        />
      </div>

      <div class="col-md-6 mb-3">
        <label
          for="cpfCnpj"
          class="form-label"
          >CPF/CNPJ:</label
        >
        <input
          id="cpfCnpj"
          :value="novoTutor.cpfCnpj"
          type="text"
          class="form-control"
          @input="atualizarCpfCnpj"
        />
        <small
          v-if="tentouEnviar && novoTutor.cpfCnpj && !cpfValido"
          class="text-danger d-block"
        >
          CPF/CNPJ inválido. Informe um valor válido ou deixe em branco.
        </small>
      </div>

      <div class="col-md-6 mb-3">
        <label
          for="cep"
          class="form-label"
          >CEP:</label
        >
        <input
          id="cep"
          :value="novoTutor.endereco.cep"
          type="text"
          class="form-control"
          @input="
            novoTutor.endereco.cep = aplicarMascaraCep($event.target.value);
            buscarCep();
          "
        />
      </div>

      <div class="col-md-6 mb-3">
        <label
          for="rua"
          class="form-label"
          >Rua:</label
        >
        <input
          id="rua"
          v-model="novoTutor.endereco.rua"
          type="text"
          class="form-control"
        />
      </div>

      <div class="col-md-3 mb-3">
        <label
          for="numero"
          class="form-label"
          >Número:</label
        >
        <input
          id="numero"
          v-model="novoTutor.endereco.numero"
          type="text"
          class="form-control"
        />
      </div>

      <div class="col-md-3 mb-3">
        <label
          for="bairro"
          class="form-label"
          >Bairro:</label
        >
        <input
          id="bairro"
          v-model="novoTutor.endereco.bairro"
          type="text"
          class="form-control"
        />
      </div>

      <div class="col-md-4 mb-3">
        <label
          for="cidade"
          class="form-label"
          >Cidade:</label
        >
        <input
          id="cidade"
          v-model="novoTutor.endereco.cidade"
          type="text"
          class="form-control"
        />
      </div>

      <div class="col-md-2 mb-3">
        <label
          for="estado"
          class="form-label"
          >Estado:</label
        >
        <input
          id="estado"
          v-model="novoTutor.endereco.estado"
          type="text"
          class="form-control"
        />
      </div>

      <div class="d-flex gap-2">
        <button
          type="submit"
          class="btn btn-success"
        >
          Salvar
        </button>
        <button
          type="button"
          class="btn btn-secondary"
          @click="router.push({ name: 'tutores' })"
        >
          Cancelar
        </button>
      </div>
    </form>
  </div>
</template>