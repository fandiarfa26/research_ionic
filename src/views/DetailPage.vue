<script setup lang="ts">
import { ProductType } from "@/product_type";
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
} from "@ionic/vue";
import { ref, watchEffect } from "vue";
import { useRoute } from "vue-router";

const route = useRoute();
const { id } = route.params;

const product = ref<ProductType | null>(null);

watchEffect(async () => {
  const response = await fetch(`https://api.restful-api.dev/objects/${id}`);
  const data = await response.json();
  product.value = data;
});
</script>

<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Detail</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <div v-if="product" class="ion-padding">
        <h1>{{ product?.name }}</h1>
        <p>Data</p>
        <ul>
          <li v-for="(value, key) in product?.data" :key="key">
            {{ key }}: {{ value }}
          </li>
        </ul>
      </div>
    </ion-content>
  </ion-page>
</template>
