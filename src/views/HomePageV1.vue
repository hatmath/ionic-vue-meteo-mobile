<template>
  <ion-page>
    <ion-header>
      <ion-toolbar color="primary">
        <ion-title>Ma météo</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content>
      <ion-item class="no-divider">
        <ion-label class="ion-text-center">{{ date }}</ion-label>
      </ion-item>
      <ion-item class="no-divider">  
        <ion-select placeholder="Choisir" v-model="ville" @ionChange="onSelectChange" label="Ville">
          <ion-select-option value="Montreal">Montréal</ion-select-option>
          <ion-select-option value="Laval">Laval</ion-select-option>
          <ion-select-option value="Quebec">Québec</ion-select-option>
          <ion-select-option value="position">Position actuelle</ion-select-option>
        </ion-select>
      </ion-item>
      <ion-item class="no-divider"><ion-label class="ion-text-center">{{ ville }}</ion-label></ion-item>
      <ion-item class="no-divider"><ion-label class="ion-text-center">{{ temperature }}</ion-label></ion-item>
      <ion-text><br></ion-text>
      <ion-item class="no-divider ion-justify-content-center">
        <ion-img :src="image" alt="Icone météo"
          style="min-width: 100px; min-height: 100px; max-width: 200px; width: 100%; height: auto; display: block; margin: 0 auto;"></ion-img>
      </ion-item>
      <ion-text><br></ion-text>
      <ion-item class="no-divider"><ion-label class="ion-text-center">{{ description }}</ion-label></ion-item>
      <div class="ion-text-center">
        <ion-button @click="afficheDialogue" v-if="false">Afficher infos</ion-button>
      </div>
    </ion-content>
    <ion-footer>
      <ion-toolbar color="secondary">
        <ion-title>© Mathieu Hatin</ion-title>
      </ion-toolbar>
    </ion-footer>
  </ion-page>
</template>
  
<script lang="ts">

import {
  IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonFooter, IonBackButton, IonButtons, IonButton,
  IonItem, IonLabel, IonInput, IonSelect, IonSelectOption, IonText, IonImg,
} from '@ionic/vue';

import moment from 'moment';
import 'moment/locale/fr';

import { alertController } from '@ionic/vue';
import { loadingController } from '@ionic/vue';
import { defineComponent } from 'vue';
import { Geolocation } from '@capacitor/geolocation';

export default defineComponent({
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonFooter,
    IonBackButton,
    IonButtons,
    IonButton,
    IonItem,
    IonLabel,
    IonInput,
    IonSelect,
    IonSelectOption,
    IonText,
    IonImg,
  },
  ionViewDidEnter() {
    console.log("Meteo page did enter");
  },
  data() {
    // let maDate = moment().locale('fr').format('dddd, D MMMM YYYY');
    // let maDate = moment().locale('fr').format('L');
    return {
      ville: '',
      date: moment().locale('fr').format('dddd, D MMMM YYYY'),
      longitude: 0,
      latitude: 0,
      temperature: "",
      image: this.getImage("BLANK_ICON", ".png", "local"),
      description: "",
    }
  },
  methods: {
    async afficheDialogue() {
      console.log("Mon alerte");
      const alert = await alertController.create({
        header: 'Vos infos météo',
        subHeader: '',
        message: "Date: " + this.date + "<br>" +
          "Ville: " + this.ville + "<br>" +
          "Position(lat/long): " + this.latitude + "/" + this.longitude,
        buttons: ['OK'],
      });
      await alert.present();
      const { role } = await alert.onDidDismiss();
      console.log('onDidDismiss: ', role);
    },
    async getWeather() {
      const loading = await loadingController.create({
        message: 'Attendre SVP ...',
      });
      await loading.present();
      let url = "";
      if (this.ville == "position") {
        const coordinates = await Geolocation.getCurrentPosition();
        console.log("Current", coordinates);
        this.latitude = coordinates.coords.latitude;
        this.longitude = coordinates.coords.longitude;
        url = `https://api.openweathermap.org/data/2.5/weather?lat=${this.latitude}&lon=${this.longitude}&APPID=6715c5f815e726411841c2550de388de&lang=fr`;
      } else {
        url = `https://api.openweathermap.org/data/2.5/weather?q=${this.ville},ca&APPID=6715c5f815e726411841c2550de388de&lang=fr`;
      }

      fetch(url)
        .then(reponse => reponse.json())
        .then(data => {
          console.log("api data:");
          console.log(data);
          this.ville = data.name;
          this.temperature = (data.main.temp - 273.15).toFixed(1) + "°C";
          this.image = this.getImage(data.weather[0].icon, ".png", "api");
          this.description = data.weather[0].description
          loading.dismiss();
        });
    },
    onSelectChange() {
      console.log('Option sélectionnée :', this.ville);
      this.getWeather();
    },
    getImage(iconName: String, extension: String, orign: String) {
      let url = "";
      if (orign == "local") {
        url = `src/assets/img/${iconName}${extension}`;
      } else if (orign == "localhost") {
        url = `http://localhost:8101/src/assets/img/${iconName}${extension}`;
      } else if (orign == "api") {
        url = `https://openweathermap.org/img/wn/${iconName}${extension}`;
      }
      console.log("Image: " + url);
      return url;
    },
  }
});

</script>
  
<style scoped>
ion-item.no-divider {
  --border-style: none;
}
</style>
