<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Products</ion-title>
        <ion-buttons slot="primary">
          <ion-button @click="addProduct">
            <ion-icon slot="icon-only" :ios="add" :md="add"></ion-icon>
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Tab 1</ion-title>
        </ion-toolbar>
      </ion-header>
      <div>
      <ion-list v-if="products">
      <ion-item v-for="item in products" :key="item.id">
        {{item.name}}
      </ion-item>
      </ion-list>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { useRouter } from 'vue-router';
import { onMounted, ref } from 'vue'
import axios from 'axios'
import { IonPage, IonHeader, IonIcon, IonToolbar, IonTitle, IonContent, IonItem, IonList, IonButton, IonButtons } from '@ionic/vue';
import { add } from 'ionicons/icons';
const router = useRouter()
const keys = ref({})
const products = ref([])

const addProduct = () => router.push('/tabs/add-product')
onMounted(async () => {
  if(localStorage.getItem("keys")) {
    keys.value = JSON.parse(localStorage.getItem('keys'))
    let url = keys.value.url + '/wp-json/wc/v3/products'
    products.value = await axios.get(url+'?consumer_key='+keys.value.ck+'&consumer_secret='+keys.value.cs).then( r => r.data)
    console.log(products.value)
  }
  else router.push('/tabs/tab3')

})

</script>
