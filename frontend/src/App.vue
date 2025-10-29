<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import { mdiPencil, mdiCheck, mdiClose } from "@mdi/js";

const API = "https://localhost:7278/api/region";

const showPopup = ref(false);
const popupMessage = ref("");
const popupType = ref("success");
const awaitingConfirmation = ref(false);
const pendingInactivationId = ref(null);
const regioes = ref([]);
const uf = ref("");
const nome = ref("");
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

const closePopup = () => {
  showPopup.value = false;
  awaitingConfirmation.value = false;
  pendingInactivationId.value = null;
};

const handlePopupConfirm = async () => {
  try {
    await axios.patch(`${API}/${pendingInactivationId.value}/inactive`);
    popupMessage.value = "Região inativada com sucesso!";
    popupType.value = "success";
    await carregarRegioes();
  } catch (error) {
    popupMessage.value =
      error.response?.data?.message || "Erro ao inativar região";
    popupType.value = "error";
  } finally {
    awaitingConfirmation.value = false;
    pendingInactivationId.value = null;
  }
};

const inativar = async (id) => {
  pendingInactivationId.value = id;
  awaitingConfirmation.value = true;
  popupType.value = "warning";
  popupMessage.value = "Deseja realmente inativar esta região?";
  showPopup.value = true;
};

const salvar = async () => {
  const regiao = {
    uf: uf.value,
    nome: nome.value,
    ...(idEditando.value && { id: idEditando.value }),
  };
  try {
    if (idEditando.value) {
      await axios.put(API, regiao);
      popupMessage.value = "Região atualizada com sucesso!";
    } else {
      await axios.post(API, regiao);
      popupMessage.value = "Região cadastrada com sucesso!";
    }
    popupType.value = "success";
    limparCampos();
    await carregarRegioes();
  } catch (error) {
    popupMessage.value =
      error.response?.data?.message || "Erro: região já existe!";
    popupType.value = "error";
  } finally {
    showPopup.value = true;
  }
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

const limparCampos = () => {
  uf.value = "";
  nome.value = "";
  idEditando.value = null;
};

const cancelarEdicao = () => {
  limparCampos();
};

onMounted(carregarRegioes);
</script>

<template>
  <div class="container">
    <!-- Pop-up dinâmico -->
    <!-- Pop-up dinâmico -->
    <div v-if="showPopup" class="popup-overlay" @click.self="closePopup">
      <div class="popup" :class="popupType">
        <p>{{ popupMessage }}</p>
        <div class="popup-buttons">
          <button
            v-if="awaitingConfirmation"
            type="button"
            @click="handlePopupConfirm"
          >
            Confirmar
          </button>
          <button type="button" @click="closePopup">
            {{ awaitingConfirmation ? "Cancelar" : "OK" }}
          </button>
        </div>
      </div>
    </div>
    <h2>Cadastro de Regiões</h2>
    <form @submit.prevent="salvar" class="form-regiao">
      <label>UF</label>
      <select v-model="uf" required>
        <option value="">Selecione</option>
        <option v-for="estado in estados" :key="estado">{{ estado }}</option>
      </select>

      <label>Região</label>
      <input
        type="text"
        v-model="nome"
        placeholder="Digite a região"
        required
      />

      <div class="botoes">
        <button type="submit">
          {{ idEditando ? "Atualizar" : "Inserir" }}
        </button>
        <button
          v-if="idEditando"
          type="button"
          class="cancelar"
          @click="cancelarEdicao"
        >
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
            {{ r.ativo ? "Ativo" : "Inativo" }}
          </td>
          <td class="acoes">
            <button class="icon-button" @click="editar(r)" title="Editar">
              <svg viewBox="0 0 24 24" width="24" height="24">
                <path :d="mdiPencil" fill="currentColor" />
              </svg>
            </button>
            <button
              v-if="r.ativo"
              class="icon-button danger"
              @click="inativar(r.id)"
              title="Inativar"
            >
              <svg viewBox="0 0 24 24" width="24" height="24">
                <path :d="mdiClose" fill="currentColor" />
              </svg>
            </button>
            <button
              v-else
              class="icon-button success"
              @click="ativar(r.id)"
              title="Ativar"
            >
              <svg viewBox="0 0 24 24" width="24" height="24">
                <path :d="mdiCheck" fill="currentColor" />
              </svg>
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap");

.container {
  max-width: 800px;
  margin: 40px auto;
  padding: 20px;
  border: 1px solid #ccc;
  background: #fff;
  border-radius: 8px;
  font-family: "Poppins", sans-serif;
  box-shadow: 0 2px 15px rgba(0, 0, 0, 0.3);
}

label{
  font-size:17px;
}

h2 {
  background: linear-gradient(135deg, #2193b0, #6dd5ed);
  padding: 15px 20px;
  border-radius: 10px;
  text-align: center;
  color: white;
  font-size: 1.9rem;
  font-weight: 600;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
  margin-bottom: 30px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  position: relative;
  overflow: hidden;
  letter-spacing: 1px;
  width: 50%;
  align-self: center;
}

h2::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(45deg, 
    rgba(255, 255, 255, 0.1) 0%,
    rgba(255, 255, 255, 0.2) 50%,
    rgba(255, 255, 255, 0) 100%);
  transform: skewX(-25deg);
}

.form-regiao {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 25px;
}

.popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.popup {
  background: white;
  padding: 20px;
  border-radius: 8px;
  min-width: 300px;
  text-align: center;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.popup p {
  margin-bottom: 25px;
  font-size: 18px;
}

.popup-buttons {
  display: flex;
  gap: 10px;
  justify-content: center;
}

.popup-buttons button {
  min-width: 100px;
  height:40px;
  padding: 8px 16px;
  border-radius: 4px;
  transition: background-color 0.2s;
}

.popup-buttons button:first-child {
  background-color: #f44336;
  /* vermelho para confirmar ação perigosa */
}

.popup-buttons button:first-child:hover {
  background-color: #d32f2f;
}

.popup-buttons button:last-child {
  background-color: #9e9e9e;
  /* cinza para cancelar/ok */
}

.popup-buttons button:last-child:hover {
  background-color: #757575;
}

.popup.success .popup-buttons button {
  background-color: #4caf50;
}

.popup.success .popup-buttons button:hover {
  background-color: #45a049;
}

/* Estilo para popup de erro */
.popup.error .popup-buttons button {
  background-color: #f44336;
}

.popup.error .popup-buttons button:hover {
  background-color: #d32f2f;
}

.popup.warning {
  border-top: 4px solid #ff9800;
}

input,
select {
  padding: 8px;
  border: 1px solid #aaa;
  border-radius: 8px;
  font-family: "Poppins", sans-serif;
  font-size: 17px;
}

button {
  width: 120px;
  height: 40px;
  padding: 8px;
  border: none;
  background: #4caf50;
  color: white;
  border-radius: 4px;
  cursor: pointer;
  font-size: 20px;

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

.tabela th,
.tabela td {
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
