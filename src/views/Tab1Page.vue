<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Products</ion-title>
        <ion-buttons slot="primary">
          <ion-button @click="openProductModal(true, null)">
            <ion-icon slot="icon-only" :ios="add" :md="add"></ion-icon>
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
    <div class="ion-padding">
      <div v-if="products">
      <h3>Products <span v-if="comp">| {{comp.name}}</span><span v-if="tag">| {{tag.name}}</span></h3>
      <ion-list>
      <ion-item v-for="item in products" :key="item.id">
        <ion-thumbnail slot="start" v-if="item.images.length > 0" @click="doConsole(item.id)">
          <img :src="item.images[0].src" />
        </ion-thumbnail>
        <ion-thumbnail slot="start" v-else @click="doConsole(item.id)">
          <img :src="noImage" />
        </ion-thumbnail>
        <ion-label class="ion-text-wrap">
          <h3>{{item.name}}</h3>
          <ion-note v-html="item.short_description"></ion-note>
          <div style="text-align:right" v-if="item.sale_price"><span style="text-decoration: line-through;">&#8377;{{item.regular_price}}</span> <span>&#8377;{{item.sale_price}}</span></div>
          <div style="text-align:right" v-else><span>&#8377;{{item.regular_price}}</span></div>
          <div style="display:flex; justify-content: end;">     
            <ion-button fill="clear" @click="openGalleryModal(true, item)" size="small">
              <ion-icon size="small" slot="icon-only" :icon="imagesOutline"></ion-icon>
            </ion-button>
            <ion-button fill="clear" @click="openProductModal(true, item)" size="small">
              <ion-icon size="small" slot="icon-only" :icon="createOutline"></ion-icon>
            </ion-button>
            <ion-button fill="clear" @click="showDelAlert(item.id)" size="small">
              <ion-icon size="small" slot="icon-only" :icon="trashOutline"></ion-icon>
            </ion-button>
        </div>   
        </ion-label>
      </ion-item>
      </ion-list>
      <div style="display:flex; justify-content: center;" v-if="pages > 1">
        <v-pagination
          v-model="page"
          :pages="pages"
          :range-size="1"
          active-color="#DCEDFF"
          @update:modelValue="getProducts"
        />
      </div>
      </div>
      <div v-if="products.length == 0" class="ion-padding">
        <ion-spinner name="crescent"></ion-spinner>
      </div>
    </div>
      <div class="ion-padding">
        <div v-if="companies.length == 0" class="ion-padding">
          <ion-spinner name="crescent"></ion-spinner>
        </div>
        <div>
          <div style="display:flex; justify-content: space-between; border-bottom: 2px solid #333">
            <h3>Companies</h3>
            <v-pagination v-if="compPages > 1"
              v-model="comPage"
              :pages="compPages"
              :range-size="1"
              active-color="#DCEDFF"
              @update:modelValue="updateCompanies"
            />
          </div>
          <ion-button size="small" fill="clear" color="primary" v-for="com in companies" :key="com.id" @click="updateProductsByCat(com)">
            <ion-label>{{com.name}}</ion-label><ion-label style="background-color:bisque; padding:5px; margin-left:5px; border-radius: 5px;">{{com.count}}</ion-label>
          </ion-button>
        </div>
      </div>
      <div class="ion-padding">
        <div v-if="tags.length == 0" class="ion-padding">
          <ion-spinner name="crescent"></ion-spinner>
        </div>
        <div>
          <div style="display:flex; justify-content: space-between; border-bottom: 2px solid #333">
            <h3>Tags</h3>
            <v-pagination v-if="tagPages > 1"
              v-model="tagPage"
              :pages="tagPages"
              :range-size="1"
              active-color="#DCEDFF"
              @update:modelValue="updateTags"
            />
          </div>
          <ion-button size="small" fill="clear" color="primary" v-for="tag in tags" :key="tag.id" outline @click="updateProductsByTag(tag)">
            <ion-label>{{tag.name}}</ion-label><ion-label style="background-color:bisque; padding:5px; margin-left:5px; border-radius: 5px;">{{tag.count}}</ion-label>
          </ion-button>
        </div>
      </div>     
    </ion-content>

    <ion-modal :is-open="productModal">
      <ion-header>
        <ion-toolbar>
          <ion-title>Modal</ion-title>
          <ion-buttons slot="end">
            <ion-button @click="openProductModal(false, null)">Close</ion-button>
          </ion-buttons>
        </ion-toolbar>
      </ion-header>
      <ion-content class="ion-padding">
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
                :options = "allCompanies"
                label="name"
                multiple
                v-model="newProduct.categories"
                taggable
                @option:created = "addCat"
                @search = "searchCat"
                append-to-body
                >
                <template v-slot:option="option">
                  <span style="color:#ccc">
                  {{ option.name }}
                  </span>
                </template>
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
                :options = "allTags"
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
                <ion-button @click="addProduct"><span v-if="newProduct.id">Update</span><span v-else>Add</span></ion-button>
              </ion-item>
            </ion-col>
          </ion-row>
          </ion-grid>
      </ion-content>
    </ion-modal>

    <ion-modal :is-open="galleryModal">
      <ion-header>
        <ion-toolbar>
          <ion-title>Modal</ion-title>
          <ion-buttons slot="end">
            <ion-button @click="openGalleryModal(false, null)">Close</ion-button>
          </ion-buttons>
        </ion-toolbar>
      </ion-header>
      <ion-content class="ion-padding">
        <ul v-if="newProduct.images.length > 0" class="image-gallery">
          <li v-for="(img, i) in newProduct.images" :key="i">
            <img :src="img.src" alt="" />
            <div class="overlay"><span><ion-button fill="clear" @click="removeImg(i)"><ion-icon slot="icon-only" :icon="closeOutline"></ion-icon></ion-button></span></div>
          </li>
        </ul>
        <h3>Add Images</h3>
        <ion-item fill="solid">
          <ion-label position="floating">Add Image</ion-label>
          <ion-input v-model="imgSrc" type="url"></ion-input>
          <ion-button @click="getImage" slot="end"><ion-icon slot="icon-only" :icon="cameraOutline"></ion-icon></ion-button>
          <ion-button @click="updateThumbnail" slot="end">Add</ion-button>
        </ion-item>
        <ion-item>
          <ion-button @click="saveImages" slot="end">Update</ion-button>
        </ion-item>
      </ion-content>
    </ion-modal>

  </ion-page>
</template>

<script setup>
import { useRouter } from 'vue-router';
import { onMounted, ref } from 'vue'
import { Camera, CameraResultType } from '@capacitor/camera'
import axios from 'axios'
import { alertController, IonPage, IonGrid, IonRow, IonNote, IonCol, IonInput, IonHeader, IonModal, IonThumbnail, IonIcon, IonToolbar, IonTitle, IonContent, IonItem, IonList, IonLabel, IonButton, IonButtons, IonSpinner } from '@ionic/vue';
import { add, cameraOutline, closeOutline, createOutline, imagesOutline, trashOutline } from 'ionicons/icons'
import vSelect from 'vue-select'
import 'vue-select/dist/vue-select.css'
import VPagination from "@hennge/vue3-pagination";
import "@hennge/vue3-pagination/dist/vue3-pagination.css";
import noImage from '../no-image.png'
const router = useRouter()
const keys = ref({})
const products = ref([])
const imgSrc = ref('')
const defaultProduct = {
    name: "",
    type: "simple",
    regular_price: "",
    sale_price:"",
    short_description: "",
    categories: [],
    tags: [],
    images: [],
  }
const newProduct = ref({})
const tags = ref([])
const companies = ref([])
const allCompanies = ref([])
const allTags = ref([])
const tag = ref()
const comp = ref()
const page = ref(1)
const pages = ref(1)
const tagPage = ref(1)
const tagPages = ref(1)
const comPage = ref(1)
const compPages = ref(1)
const getProducts = async() => {
  products.value = []
  let t = (tag.value) ? '&tag='+tag.value.id : ''
  let c = (comp.value) ? '&category='+comp.value.id : ''
  let p = (page.value) ? '&page='+page.value : ''
  let url = `${keys.value.url}/wp-json/wc/v3/products?per_page=10${t}${c}${p}`
  let auth = {username: keys.value.ck, password: keys.value.cs}
  let res = await axios.get(url, {auth: auth})
  products.value = res.data
  console.log(products.value)
  pages.value = +res.headers["x-wp-totalpages "]
}

const updateProductsByCat = (cat) => {
  comp.value = cat
  getProducts()
}

const updateProductsByTag = (t) => {
  tag.value = t
  getProducts()
}

const updateCompanies = async() => {
  let url = keys.value.url + '/wp-json/wc/v3/'
  let auth = {username: keys.value.ck, password: keys.value.cs}
  let cr = await axios.get(url+'products/categories?per_page=10&page='+comPage.value, {auth: auth})
  companies.value = cr.data
}

const updateTags = async() => {
  let url = keys.value.url + '/wp-json/wc/v3/'
  let auth = {username: keys.value.ck, password: keys.value.cs}
  let cr = await axios.get(url+'products/tags?per_page=10&page='+tagPage.value, {auth: auth})
  tags.value = cr.data
}

const productModal = ref(false)
const openProductModal = (open, item) => {
  productModal.value = open
  newProduct.value = (item) ? item : defaultProduct
}
const galleryModal = ref(false)
const openGalleryModal = (open, item) => {
  galleryModal.value = open
  newProduct.value = (item) ? item : defaultProduct
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
    imgSrc.value = up.data.data.url
  }


const updateThumbnail = () => {
  newProduct.value.images.push({src: imgSrc.value})
  imgSrc.value = ''
}

const removeImg = (i) => {
  newProduct.value.images.splice(i, 1)
}

const saveImages = async() => {
  let url = `${keys.value.url}/wp-json/wc/v3/products`
  let auth = {username: keys.value.ck, password: keys.value.cs}
  let res = await axios.put(url+'/'+newProduct.value.id, newProduct.value, { auth: auth})
     if(res.data) {
      galleryModal.value = false
      newProduct.value = defaultProduct
     }
}

const showDelAlert = async (id) => {
  const alert = await alertController.create({
          header: 'Alert!',
          subHeader: 'Do you really want to delete this?',
          buttons: [
            {
              text: 'Cancel',
              role: 'cancel',
            },
            {
              text: 'Yes',
              role: 'confirm',
              handler: async () => {
                let url = `${keys.value.url}/wp-json/wc/v3/products/${id}`
                let res = await axios.delete(url, { auth: {username: keys.value.ck, password: keys.value.cs}})
                if(res.data) products.value.splice(products.value.findIndex(item => item.id == id), 1)
              },
            },
          ],
        });

        await alert.present();
}

const searchCat = async(str, loading) => {
  if(str.length > 1){
    loading(true)
    let url = `${keys.value.url}/wp-json/wc/v3/products/categories?search=${str.toUpperCase()}`
    let res = await axios.get(url, { auth: {username: keys.value.ck, password: keys.value.cs}})
    allCompanies.value = await res.data
    loading(false)
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

const searchTag = async(str, loading) => {
  if(str.length > 1){
    loading(true)
    let url = `${keys.value.url}/wp-json/wc/v3/products/tags?search=${str.toUpperCase()}`
    let res = await axios.get(url, { auth: {username: keys.value.ck, password: keys.value.cs}})
    allTags.value = await res.data
    loading(false)
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

const addProduct = async() => {
  let url = `${keys.value.url}/wp-json/wc/v3/products`
  let auth = {username: keys.value.ck, password: keys.value.cs}
  if(newProduct.value.id){
    try {
     let res = await axios.put(url+'/'+newProduct.value.id, newProduct.value, { auth: auth})
     if(res.data) productModal.value = false
    }
    catch(e){
      console.log(e)
    }
  }
  else{
    try {
     let res = await axios.post(url, newProduct.value, { auth: auth})
     if(res.data) {
      productModal.value = false
      products.value.push(res.data)
     }
    }
    catch(e){
      console.log(e)
    }
  }
  console.log(newProduct.value)
}

const doConsole = (id) => console.log(id)
onMounted(async () => {
  if(localStorage.getItem("keys")) {
    keys.value = JSON.parse(localStorage.getItem('keys'))
    getProducts()
    let url = keys.value.url + '/wp-json/wc/v3/'
    let auth = {username: keys.value.ck, password: keys.value.cs}
    let tr = await axios.get(url+'products/tags?hide_empty=1&per_page=10', {auth: auth})
    tags.value = tr.data
    tagPages.value = +tr.headers["x-wp-totalpages"]
    let cr = await axios.get(url+'products/categories?hide_empty=1&per_page=10', {auth: auth})
    companies.value = cr.data
    compPages.value = +cr.headers["x-wp-totalpages"]
    console.log(compPages.value)
  }
  else router.push('/tabs/tab3')

})

</script>
<style>
ul {
  list-style: none;
}

.image-gallery {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.image-gallery > li {
  flex: 1 1 auto; /* or flex: auto; */
  height: 80px;
  cursor: pointer;
  position: relative;
}

.image-gallery::after {
  content: "";
  flex-grow: 999;
}

.image-gallery li img {
  object-fit: cover;
  width: 100%;
  height: 100%;
  vertical-align: middle;
  border-radius: 5px;
}

.overlay {
  position: absolute;
  width: 100%;
  height: 100%;
  background: rgba(57, 57, 57, 0.502);
  top: 0;
  left: 0;
  transform: scale(0);
  transition: all 0.2s 0.1s ease-in-out;
  color: #fff;
  border-radius: 5px;
  /* center overlay content */
  display: flex;
  align-items: center;
  justify-content: center;
}

/* hover */
.image-gallery li:hover .overlay {
  transform: scale(1);
}
</style>
