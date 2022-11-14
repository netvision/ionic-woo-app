<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Settings</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Tab 3</ion-title>
        </ion-toolbar>
      </ion-header>
      <ion-list>
        <ion-item>
          <ion-label position="stacked">Woo Store URL</ion-label>
          <ion-input type="url" v-model="keys.url"></ion-input>
        </ion-item>
        <ion-item>
          <ion-label position="stacked">Client Key</ion-label>
          <ion-input v-model="keys.ck"></ion-input>
        </ion-item>
        <ion-item>
          <ion-label position="stacked">Client Secret</ion-label>
          <ion-input v-model="keys.cs"></ion-input>
        </ion-item>
        <ion-item>
          <ion-button fill="outline" @click="saveKeys">Save</ion-button>
        </ion-item>
      </ion-list>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonList, IonItem, IonButton, IonInput, IonLabel } from '@ionic/vue';
import { onMounted, ref } from 'vue'
import { useRouter } from 'vue-router';

const router = useRouter()
const keys = ref({})

const saveKeys = () => {
  localStorage.setItem('keys', JSON.stringify(keys.value))
  router.push('/tabs/tab1')
}

onMounted(() => {
  if(localStorage.getItem('keys')){
    keys.value = JSON.parse(localStorage.getItem('keys'))
  }
})


</script>
