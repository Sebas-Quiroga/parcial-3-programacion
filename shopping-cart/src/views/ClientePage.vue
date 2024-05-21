<template>
  <ion-page @ionViewDidEnter="findAllRecords">
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Carro</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content fullscreen>
      <div id="container">
        <div class="nuevo-registro">
          <ion-button @click="openModalAdd"><ion-icon :icon="IonIcons.addSharp"></ion-icon></ion-button>
        </div>

        <!-- Modal para agregar o editar registros -->
        <ion-modal :is-open="modalIsOpen" @didDismiss="modalIsOpen = false" :css-class="['my-custom-modal']">
          <div >
            
            <div id="data-form">
              <InputComponent v-model="marca" id="marca" name="marca" label="Marca: " />
              <InputComponent v-model="modelo" id="modelo" name="modelo" label="Modelo: " />
              <InputComponent v-model="placa" id="placa" name="placa" label="Placa: " />
              <InputComponent v-model="color" id="color" name="color" label="Color: " />
              <InputComponent v-model="usuarioId" id="usuarioId" name="usuarioId" label="Usuario ID: " />
              <InputComponent v-model="categoriaId" id="categoriaId" name="categoriaId" label="Categoría ID: " />
            </div>
            
            <div>
              <!-- Botones de CRUD dentro del modal -->
              <CrudButtonComponent :showFind="showFind" :showCreated="showCreated" :showUpdated="showUpdated"
                :showDeleted="showDeleted" @findAllRecords="findAllRecords" @createRecord="createRecord"
                @updateRecord="updateRecord" @deleteRecordPhysical="deleteRecordPhysical" />
            </div>
          </div>
        </ion-modal>


        <!-- Mostrar datos -->
        <div class="table-container">
          <ion-list>
            <!-- Iterar sobre items para mostrar los datos en tarjetas -->
            <ion-card v-for="(item, index) in items" :key="index" class="card-width">
              <ion-card-content>
                <ion-item>
                  <ion-label>Marca: </ion-label>
                  <ion-label>{{ item.marca }}</ion-label>
                </ion-item>
                <ion-item>
                  <ion-label>Modelo: </ion-label>
                  <ion-label>{{ item.modelo }}</ion-label>
                </ion-item>
                <ion-item>
                  <ion-label>Placa: </ion-label>
                  <ion-label>{{ item.placa }}</ion-label>
                </ion-item>
                <ion-item>
                  <ion-label>Color: </ion-label>
                  <ion-label>{{ item.color }}</ion-label>
                </ion-item>
                <ion-item>
                  <ion-label>Usuario ID: </ion-label>
                  <ion-label>{{ item.usuarioId }}</ion-label>
                </ion-item>
                <ion-item>
                  <ion-label>Categoría ID: </ion-label>
                  <ion-label>{{ item.categoriaId }}</ion-label>
                </ion-item>
              </ion-card-content>
              <ion-item>
                <!-- Botones de editar y eliminar -->
                <ion-button @click="findById(item.id)">
                  <ion-icon :icon="IonIcons.createOutline"></ion-icon>
                </ion-button>
                <ion-button @click="deleteRecordPhysical(item.id)">
                  <ion-icon :icon="IonIcons.trash"></ion-icon>
                </ion-button>
              </ion-item>
            </ion-card>
          </ion-list>
        </div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonIcon, IonModal } from '@ionic/vue';
import InputComponent from '@/components/InputComponent.vue';
import CrudButtonComponent from '@/components/CrudButtonComponent.vue';
import { showSuccessMessage, showErrorMessage } from '../utils/alerts.js';
import { onMounted, ref } from 'vue';
import * as IonIcons from 'ionicons/icons';
import axios from 'axios';


const baseURL =  'http://localhost:9000/estructura/api/carro'; 
const modalIsOpen = ref(false);
const items = ref<Array<ItemType>>([]);
const id = ref('');
const marca = ref('');
const modelo = ref('');
const placa = ref('');
const color = ref('');
const usuarioId = ref('');
const categoriaId = ref('');

const showFind = ref<boolean>(false);
const showCreated = ref<boolean>(true);
const showUpdated = ref<boolean>(false);
const showDeleted = ref<boolean>(false);


// Tipos
interface ItemType {
  id: string;
  marca: string;
  modelo: string;
  placa: string;
  color: string;
  usuarioId: string;
  categoriaId: string;
}

onMounted(() => {
  findAllRecords();
});

// Métodos
async function findAllRecords() {
  try {
    const response = await axios.get(baseURL);
    items.value = response.data;
  } catch (error) {
    console.error('Error al obtener todos los registros:', error);
    throw error;
  }
}

//Cargar los datos para edición
async function findById(recordId: string) {
  try {
    await openModalAdd();
    const response = await axios.get(`${baseURL}/${recordId}`);
    const data = response.data;

    id.value = data.id;
    marca.value = data.marca;
    modelo.value = data.modelo;
    placa.value = data.placa;
    color.value = data.color;
    

    // Controlar la visibilidad de los botones
    showCreated.value = false;
    showUpdated.value = true;


  } catch (error) {
    console.error('Error al encontrar el registro por ID:', error);
    throw error;
  }
}

async function createRecord() {
  const data = {
    marca: marca.value,
    modelo: modelo.value,
    placa: placa.value,
    color: color.value,
  };

  try {
    const response = await axios.post(baseURL, data);
    console.log('Registro creado exitosamente:', response.data);
    await findAllRecords();
    await clearData();
    await showSuccessMessage();
    await closeModal();
  } catch (error) {
    console.error('Error al crear el registro:', error);
    await showErrorMessage();
  }
}

async function updateRecord() {
  const data = {
    id: id.value,
    marca: marca.value,
    modelo: modelo.value,
    placa: placa.value,
    color: color.value,
    
  };

  try {
    const response = await axios.put(`${baseURL}/${data.id}`, data);
    console.log('Registro actualizado exitosamente:', response.data, id);

    await findAllRecords();
    await clearData();

    showCreated.value = true;
    showUpdated.value = false;

    await closeModal();

    return response.data;
  } catch (error) {
    console.error('Error al actualizar el registro:', error);
    throw error;
  }
}


async function deleteRecordPhysical(id: String) {
  try {
    const response = await axios.delete(`${baseURL}/${id}`);
    console.log('Registro eliminado exitosamente:', response.data);
    await findAllRecords();
  } catch (error) {
    console.error('Error al eliminar el registro físico:', error);
    showErrorMessage();
  }
}

async function deleteRecordLogical(id) {
  try {
    const response = await axios.delete(`${baseURL}/${id}`);
    console.log('Registro eliminado exitosamente:', response.data);
    await findAllRecords();
  } catch (error) {
    console.error('Error al eliminar el registro físico:', error);
    showErrorMessage();
  }
}

async function clearData() {
  marca.value = '';
  modelo.value = '';
  placa.value = '';
  color.value = '';
  
}


const openModalAdd = () => {
  modalIsOpen.value = true;
};

const closeModal = () => {
  modalIsOpen.value = false;
};
</script>

<style scoped src="../theme/container.css"></style>