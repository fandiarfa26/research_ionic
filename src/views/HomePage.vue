<script setup lang="ts">
import { ProductType } from "@/product_type";
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonList,
  IonItem,
  IonLabel,
  IonIcon,
  IonFab,
  IonFabButton,
  IonModal,
  IonInput,
  IonButtons,
  IonButton,
  IonToast,
} from "@ionic/vue";
import { add } from "ionicons/icons";
import { ref, watchEffect } from "vue";

const items = ref<ProductType[]>([]);

const name = ref<string>("");
const toastMessage = ref<string>("");
const isToastOpen = ref<boolean>(false);
const isModalOpen = ref<boolean>(false);
const choosedId = ref<string>("");

function openCreateModal() {
  isModalOpen.value = true;
  choosedId.value = "";
  name.value = "";
}

function openEditModal(product: ProductType) {
  isModalOpen.value = true;
  choosedId.value = product.id;
  name.value = product.name;
}

function submitForm() {
  if (name.value === "") {
    toastMessage.value = "Please fill in all fields";
    isToastOpen.value = true;
    return;
  }

  if (choosedId.value !== "") {
    // edit
    fetch(`https://api.restful-api.dev/objects/${choosedId.value}`, {
      method: "PUT",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        name: name.value,
        data: {
          description: "description product",
        },
      }),
    })
      .then((response) => {
        if (!response.ok) {
          throw new Error(response.statusText);
        }
        return response.json();
      })
      .then((data) => {
        const index = items.value.findIndex(
          (item) => item.id === choosedId.value
        );
        items.value[index] = data;
        isModalOpen.value = false;
        toastMessage.value = "Product updated successfully";
        isToastOpen.value = true;
      })
      .catch((error) => {
        console.error(error);
        toastMessage.value = "Error updating product";
        isToastOpen.value = true;
      });
  } else {
    // create
    fetch("https://api.restful-api.dev/objects", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        name: name.value,
        data: {
          description: "description product",
        },
      }),
    })
      .then((response) => {
        if (!response.ok) {
          throw new Error(response.statusText);
        }
        return response.json();
      })
      .then((data) => {
        console.log(data);
        items.value.push(data);
        isModalOpen.value = false;
        toastMessage.value = "Product created successfully";
        isToastOpen.value = true;
      })
      .catch((error) => {
        console.error(error);
        toastMessage.value = "Error creating product";
        isToastOpen.value = true;
      });
  }
}

function deleteProduct(id: string) {
  fetch(`https://api.restful-api.dev/objects/${id}`, {
    method: "DELETE",
  })
    .then((response) => {
      if (!response.ok) {
        throw new Error(response.statusText);
      }
      return response.json();
    })
    .then((data) => {
      console.log(data);
      const index = items.value.findIndex(
        (item) => item.id === choosedId.value
      );
      items.value.splice(index, 1);
      toastMessage.value = "Product deleted successfully";
      isToastOpen.value = true;
    })
    .catch((error) => {
      console.error(error);
      toastMessage.value = "Error deleting product";
      isToastOpen.value = true;
    });
}

watchEffect(async () => {
  const response = await fetch("https://api.restful-api.dev/objects");
  const data = await response.json();
  console.log(data);
  items.value.push(...data);
});
</script>

<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Products</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Blank</ion-title>
        </ion-toolbar>
      </ion-header>

      <ion-list>
        <ion-item v-for="(item, index) in items" :key="index">
          <ion-label>{{ item.name }}</ion-label>
          <ion-button
            slot="end"
            size="small"
            @click="$router.push({ name: 'detail', params: { id: item.id } })"
            >Detail</ion-button
          >
          <ion-button
            slot="end"
            size="small"
            color="warning"
            @click="openEditModal(item)"
            >Edit</ion-button
          >
          <ion-button
            slot="end"
            size="small"
            color="danger"
            @click="deleteProduct(item.id)"
            >Delete</ion-button
          >
        </ion-item>
      </ion-list>
    </ion-content>

    <ion-fab slot="fixed" vertical="bottom" horizontal="end">
      <ion-fab-button @click="openCreateModal">
        <ion-icon :icon="add"></ion-icon>
      </ion-fab-button>
    </ion-fab>

    <ion-modal :is-open="isModalOpen">
      <ion-header>
        <ion-toolbar>
          <ion-title
            >{{ choosedId !== "" ? "Edit" : "Create" }} Product</ion-title
          >
          <ion-buttons slot="end">
            <ion-button @click="isModalOpen = false">Close</ion-button>
          </ion-buttons>
        </ion-toolbar>
      </ion-header>
      <ion-content class="ion-padding">
        <ion-list>
          <ion-item>
            <ion-input
              label="Name"
              label-placement="stacked"
              placeholder="Enter product name"
              v-model="name"
            ></ion-input>
          </ion-item>
        </ion-list>
        <div class="ion-padding">
          <ion-button expand="block" @click="submitForm">Submit</ion-button>
        </div>
      </ion-content>
    </ion-modal>

    <ion-toast
      :is-open="isToastOpen"
      :message="toastMessage"
      :duration="5000"
      @didDismiss="isToastOpen = false"
    ></ion-toast>
  </ion-page>
</template>

<style scoped></style>
