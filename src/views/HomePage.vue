<template>
  <ion-page>
    <ion-header>
      <ion-toolbar color="medium">
        <ion-title>Ma météo</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :style="backgroundStyle" class="ion-text-center">
      <ion-text><br></ion-text>
      <ion-label class="custom-label-nobackground">{{ date }}</ion-label>
      <ion-text><br></ion-text>
      <ion-select placeholder="Choisir" v-model="ville" @ionChange="onSelectChange" label="Ville" class="custom-label-nobackground">
        <ion-select-option value="Montreal">Montréal</ion-select-option>
        <ion-select-option value="Laval">Laval</ion-select-option>
        <ion-select-option value="Quebec">Québec</ion-select-option>
        <ion-select-option value="position">Position actuelle</ion-select-option>
      </ion-select>
      <ion-text><br></ion-text>
      <ion-label class="ion-text-center custom-label-withbackground" :class="{ 'no-padding': !ville }">{{ ville }}</ion-label>
      <ion-text><br></ion-text>
      <ion-label class="ion-text-center custom-label-withbackground" :class="{ 'no-padding': !ville }">{{ temperature }}</ion-label>
      <ion-text><br></ion-text>
      <ion-img :src="image" alt="Icone météo"
        style="min-width: 100px; min-height: 100px; max-width: 200px; width: 100%; height: auto; display: block; margin: 0 auto;">
      </ion-img>
      <ion-text><br></ion-text>
      <ion-label class="ion-text-center custom-label-withbackground" :class="{ 'no-padding': !ville }">{{ description }}</ion-label>
    </ion-content>
    <ion-footer>
      <ion-toolbar color="dark">
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
    console.log("météo -> 'view did enter'");
  },
  data() {

    return {
      ville: '',
      date: this.getFormattedFrenchDate(),
      longitude: 0,
      latitude: 0,
      temperature: "",
      image: this.getImage("blank_icon", ".png", "local"),
      description: "",
      backgroundStyle: {
        '--background': 'url(src/assets/img/b1.jpg)', // Remplacez ceci par votre chemin d'image réel
      },
    }

  },
  methods: {

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
          console.log("données météo -> ");
          console.log(data);
          this.ville = data.name;
          this.temperature = (data.main.temp - 273.15).toFixed(1) + "°C";
          this.image = this.getImage(data.weather[0].icon, ".png", "api");
          this.description = data.weather[0].description
          loading.dismiss();
        });

    },

    onSelectChange() {

      console.log('option sélectionnée -> ', this.ville);
      this.getWeather();

    },

    getImage(iconName: string, extension: string, orign: string) {
      
      let url = "";
      if (orign == "local") {
        url = `src/assets/img/${iconName}${extension}`;
      } else if (orign == "localhost") {
        url = `http://localhost:8101/src/assets/img/${iconName}${extension}`;
      } else if (orign == "api") {
        url = `https://openweathermap.org/img/wn/${iconName}${extension}`;
      }
      console.log("img -> " + url);
      this.changeBackgroundImage(new Date().getDay());
      return url;

    },

    getFormattedFrenchDate() {

      const currentDate = new Date();
      const frenchDaysOfWeek = ['Dimanche', 'Lundi', 'Mardi', 'Mercredi', 'Jeudi', 'Vendredi', 'Samedi'];
      const frenchMonths = ['Janvier', 'Février', 'Mars', 'Avril', 'Mai', 'Juin', 'Juillet', 'Août', 'Septembre', 'Octobre', 'Novembre', 'Décembre'];

      const dayOfWeek = frenchDaysOfWeek[currentDate.getDay()];
      const day = currentDate.getDate();
      const month = frenchMonths[currentDate.getMonth()];
      const year = currentDate.getFullYear();

      const formattedDate = `${dayOfWeek}, ${day} ${month} ${year}`;      
      return formattedDate;

    },

    changeBackgroundImage(dayOfWeek: number) {

      console.log("jour -> " + dayOfWeek);
      this.backgroundStyle = {
        '--background': `url(src/assets/img/b${dayOfWeek}.jpg) no-repeat`,      
      };

    },

  }
  
});

</script>
  
<style scoped>
ion-item.no-divider {
  --border-style: none;
}

ion-select {
  --padding-start: 16px;
  --padding-end: 16px;
}

.custom-label-withbackground {
  background-color: rgb(235, 226, 226);
  color: rgb(0, 0, 0);
  padding: 4px 8px;
}

.custom-label-nobackground {
  color: rgb(0, 0, 0);
}

.no-padding {
  padding: 0; /* Supprimez le padding lorsque la classe 'no-padding' est présente */
}

</style>
