<template>
    <ion-page>
      <ion-header>
        <ion-toolbar>
          <ion-buttons slot="start">
            <ion-back-button></ion-back-button>
          </ion-buttons>
          <ion-title>New Product</ion-title>
        </ion-toolbar>
      </ion-header>
      <ion-content :fullscreen="true">
      <ion-grid>
      <ion-row>
        <ion-col>
          <ion-item>
            <ion-label position="stacked">Name</ion-label>
            <ion-input type="url" v-model="newProduct.name"></ion-input>
          </ion-item>
        </ion-col>
      </ion-row>
      <ion-row>
        <ion-col>
          <ion-item>
          <ion-label position="stacked" style="margin-bottom:5px">Company: </ion-label>
            <v-select
            placeholder="Select Company"
            :options = "options"
            label="name"
            multiple
            v-model="newProduct.categories"
            taggable
            @option:created = "addCat"
            @search = "searchCat"
            append-to-body
            >
            <template v-slot:no-options="{ search, searching }">
                <template v-if="searching">
                  No results found for <em>{{ search }}</em>. Click to create!
                </template>
                <em v-else style="opacity: 0.5">Start typing to search for a company.</em>
          </template>
            </v-select>
          </ion-item>
          </ion-col>
          <ion-col>
          <ion-item>
          <ion-label position="stacked" style="margin-bottom:5px">Tags: </ion-label>
            <v-select
            :options = "tags"
            label="name"
            multiple
            v-model="newProduct.tags"
            taggable
            @option:created = "addTag"
            @search = "searchTag"
            append-to-body
            >
            <template v-slot:no-options="{ search, searching }">
                <template v-if="searching">
                  No results found for <em>{{ search }}</em>. Click to create!
                </template>
                <em v-else style="opacity: 0.5">Start typing to search.</em>
          </template>
            </v-select>
          </ion-item>
          </ion-col>
          </ion-row>
          <ion-row>
          <ion-col>
          <ion-item>
            <ion-label position="stacked">Price: </ion-label>
            <ion-input v-model="newProduct.regular_price"></ion-input>
          </ion-item>
        </ion-col>
        <ion-col>
          <ion-item>
            <ion-label position="stacked">Sale Price: </ion-label>
            <ion-input v-model="newProduct.sale_price"></ion-input>
          </ion-item>
        </ion-col>
      </ion-row>
      <ion-row>
        <ion-col>
          <ion-item>
            <ion-label position="stacked">Description</ion-label>
            <ion-input v-model="newProduct.short_description"></ion-input>
          </ion-item>
        </ion-col>
      </ion-row>
      <ion-row>
        <ion-col>
          <ion-item>
            <ion-button @click="getImage">Select Image</ion-button>
          </ion-item>
        </ion-col>
      </ion-row>
      <ion-row>
        <ion-col>
          <ion-item>
            <ion-button @click="addProduct">Add</ion-button>
          </ion-item>
        </ion-col>
      </ion-row>
      </ion-grid>
        
        <pre>{{imgs}}</pre>
        <pre>{{photoPath}}</pre>
        <pre>{{newProduct}}</pre>
        <pre>{{addedProduct}}</pre>
        <pre>{{errors}}</pre>
      </ion-content>
    </ion-page>
  </template>
  <script setup>
  //import { ImagePicker } from '@ionic-native/image-picker'
  //import { Capacitor } from '@capacitor/core'
  import { Camera, CameraResultType } from '@capacitor/camera'
  //import { Filesystem, Directory } from '@capacitor/filesystem'
  import axios from "axios"
  
  import { ref } from 'vue';
  import vSelect from 'vue-select'
  import 'vue-select/dist/vue-select.css'

  import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonButtons, IonBackButton, IonButton, IonGrid, IonRow, IonCol, IonItem, IonLabel, IonInput } from '@ionic/vue';
  
  const options = ref([])
  const tags = ref([])
  const imgs = ref({})
  const photoPath = ref()
  const errors = ref({})
  const addedProduct = ref({})
  const newProduct = ref({
    name: "",
    type: "simple",
    regular_price: "",
    short_description: "",
    categories: [],
    tags: [],
    images: [],
  })

  const keys = ref(JSON.parse(localStorage.getItem('keys')))

  const addProduct = async() => {
    if(newProduct.value.cats){
        newProduct.value.cats.forEach(cat => {
          let category = { id: cat.id }
          newProduct.value.categories.push(category)
      })
        
    }
    delete newProduct.value.cats
    
    let url = `${keys.value.url}/wp-json/wc/v3/products`
    try {
     addedProduct.value = await axios.post(url, newProduct.value, { auth: {username: keys.value.ck, password: keys.value.cs}}).then(r => r.data)
    }
    catch(e){
      console.log(e)
      errors.value = e
    }
  }
  
  const addCat = async (opt) => {
    console.log(opt)
    let url = `${keys.value.url}/wp-json/wc/v3/products/categories`
    try {
      //console.log(newProduct.value.categories, opt)
      let res = await axios.post(url, {name: opt.toUpperCase()}, { auth: {username: keys.value.ck, password: keys.value.cs}})
      newProduct.value.categories.push(res.data)
      newProduct.value.categories.splice(newProduct.value.categories.findIndex(item => !item.id), 1)
    }
    catch(e){
      console.log(e.message)
    }
  }

  const addTag = async (opt) => {
    console.log(opt)
    let url = `${keys.value.url}/wp-json/wc/v3/products/tags`
    try {
      //console.log(newProduct.value.categories, opt)
      let res = await axios.post(url, {name: opt.toUpperCase()}, { auth: {username: keys.value.ck, password: keys.value.cs}})
      newProduct.value.tags.push(res.data)
      newProduct.value.tags.splice(newProduct.value.tags.findIndex(item => !item.id), 1)
    }
    catch(e){
      console.log(e.message)
    }
  }

 const searchCat = async(str, loading) => {
  if(str.length > 1){
    loading(true)
    let url = `${keys.value.url}/wp-json/wc/v3/products/categories?search=${str.toUpperCase()}`
    let res = await axios.get(url, { auth: {username: keys.value.ck, password: keys.value.cs}})
    options.value = await res.data
    loading(false)
  }
 }

 const searchTag = async(str, loading) => {
  if(str.length > 1){
    loading(true)
    let url = `${keys.value.url}/wp-json/wc/v3/products/tags?search=${str.toUpperCase()}`
    let res = await axios.get(url, { auth: {username: keys.value.ck, password: keys.value.cs}})
    tags.value = await res.data
    loading(false)
  }
 }

const getImage = async() => {
    let pics = await Camera.getPhoto({
      quality: 60,
      allowEditing: true,
      width: 600,
      resultType: CameraResultType.Base64
    })
    let up = await axios.post(
      'https://api.imgbb.com/1/upload', 
      {
        key: 'a9e41e92100f265146238ced4aaf6a2c', 
        image: pics.base64String,
        expiration: 600
      }, 
      { 
        headers: 
        {
          'Content-Type': 'multipart/form-data'
          }
      }
    )
    imgs.value = up.data.data.url
    newProduct.value.images.push({src: up.data.data.url})
  }
</script>
  