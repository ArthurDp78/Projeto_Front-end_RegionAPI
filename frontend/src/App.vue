<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';
import { mdiPencil, mdiCheck, mdiClose } from '@mdi/js';

// URL base da sua API .NET
const API = 'https://localhost:7278/api/region';

const regioes = ref([]);
const uf = ref('');
const nome = ref('');
const idEditando = ref(null);

// lista de estados
const estados = [
  'AC','AL','AP','AM','BA','CE','DF','ES','GO','MA','MG','MS','MT',
  'PA','PB','PE','PI','PR','RJ','RN','RO','RR','RS','SC','SE','SP','TO'
];

const carregarRegioes = async () => {
  const res = await axios.get(API);
  regioes.value = res.data.sort(
    (a, b) => a.uf.localeCompare(b.uf) || a.nome.localeCompare(b.nome)
  );
};

const salvar = async () => {
  const regiao = { 
    uf: uf.value, 
    nome: nome.value,
    ...(idEditando.value && { id: idEditando.value })
  };
  if (idEditando.value) {
    await axios.put(API, regiao);
  } else {
    await axios.post(API, regiao);
  }
  limparCampos();
  await carregarRegioes();
};

const editar = (r) => {
  uf.value = r.uf;
  nome.value = r.nome;
  idEditando.value = r.id;
};

const ativar = async (id) => {
  await axios.patch(`${API}/${id}/active`);
  await carregarRegioes();
};

const inativar = async (id) => {
  if (confirm('Deseja realmente inativar esta região?')) {
    await axios.patch(`${API}/${id}/inactive`);
    await carregarRegioes();
  }
};

const limparCampos = () => {
  uf.value = '';
  nome.value = '';
  idEditando.value = null;
};

const cancelarEdicao = () => {
  limparCampos();
};

onMounted(carregarRegioes);
</script>


<template>
  <div class="container">
    <h2>Cadastro de Regiões</h2>
    <form @submit.prevent="salvar" class="form-regiao">
      <label >UF</label>
      <select v-model="uf" required>
        <option value="">Selecione</option>
        <option v-for="estado in estados" :key="estado">{{ estado }}</option>
      </select>

      <label>Região</label>
      <input type="text" v-model="nome" placeholder="Digite a região" required />

      <div class="botoes">
        <button type="submit">
          {{ idEditando ? 'Atualizar' : 'Inserir' }}
        </button>
        <button v-if="idEditando" type="button" class="cancelar" @click="cancelarEdicao">
          Cancelar
        </button>
      </div>
    </form>

    <table class="tabela">
      <thead>
        <tr>
          <th>UF</th>
          <th>Região</th>
          <th>Situação</th>
          <th>Ações</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="r in regioes" :key="r.id">
          <td>{{ r.uf }}</td>
          <td>{{ r.nome }}</td>
          <td :class="r.ativo ? 'ativo' : 'inativo'">
            {{ r.ativo ? 'Ativo' : 'Inativo' }}
          </td>
          <td class="acoes">
            <button class="icon-button" @click="editar(r)" title="Editar">
              <svg viewBox="0 0 24 24" width="24" height="24">
                <path :d="mdiPencil" fill="currentColor"/>
              </svg>
            </button>
            <button v-if="r.ativo" class="icon-button danger" @click="inativar(r.id)" title="Inativar">
              <svg viewBox="0 0 24 24" width="24" height="24">
                <path :d="mdiClose" fill="currentColor"/>
              </svg>
            </button>
            <button v-else class="icon-button success" @click="ativar(r.id)" title="Ativar">
              <svg viewBox="0 0 24 24" width="24" height="24">
                <path :d="mdiCheck" fill="currentColor"/>
              </svg>
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style scoped>
.container {
  max-width: 800px;
  margin: 40px auto;
  padding: 20px;
  border: 1px solid #ccc;
  background: #fff;
  border-radius: 8px;
  font-family: 'Poppins', sans-serif;
}

h2 {
  background: linear-gradient(135deg, #f6b26b, #e69138);
  padding: 10px;
  border-radius: 5px;
  text-align: center;
  color: white;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.2);
}

.form-regiao {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 25px;
}

input, select {

  padding: 8px;
  border: 1px solid #aaa;
  border-radius: 8px;
}

button {
  width: 120px;
  padding: 8px;
  border: none;
  background: #4caf50;
  color: white;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background: #45a049;
}

.botoes {
  display: flex;
  gap: 10px;
  justify-content: center;
}

button.cancelar {
  background: #f44336;
}

button.cancelar:hover {
  background: #d32f2f;
}

.tabela {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0;
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.tabela th, .tabela td {
  border: none;
  border-bottom: 1px solid #ddd;
  border-right: 1px solid #ddd;
  padding: 12px;
  text-align: center;
  background: white;
}

.tabela th {
  background: #f8f9fa;
  font-weight: 600;
}

.tabela tr:last-child td {
  border-bottom: none;
}

.tabela th:last-child,
.tabela td:last-child {
  border-right: none;
}

.ativo {
  color: rgb(16, 199, 16);
  font-weight: bold;
}

.inativo {
  color: rgb(212, 16, 16);
  font-weight: bold;
}

a {
  color: blue;
  cursor: pointer;
}

.acoes {
  display: flex;
  gap: 8px;
  justify-content: center;
  align-items: center;
}

.icon-button {
  width: 36px;
  height: 36px;
  padding: 6px;
  border: none;
  background: #f0f0f0;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  color: #666;
}

.icon-button:hover {
  background: #e0e0e0;
  transform: scale(1.1);
}

.icon-button.danger {
  background: #ffebee;
  color: #d32f2f;
}

.icon-button.danger:hover {
  background: #ffcdd2;
}

.icon-button.success {
  background: #e8f5e9;
  color: #2e7d32;
}

.icon-button.success:hover {
  background: #c8e6c9;
}
</style>
