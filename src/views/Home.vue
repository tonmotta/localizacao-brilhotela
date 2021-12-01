<template>
  <ion-page>
    <ion-header :translucent="true">
      
      <ion-toolbar>
            <ion-button slot="end" style="margin-right: 3%;" colour="dark" @click="aumentarBrilho()">
<ion-icon src="brilho.svg" size="small">
        </ion-icon>
            </ion-button>
        <ion-title>Registro de Localização</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <div class='container' align='center'>
          <h1>Suas coordenadas atuais são</h1>
          <p>Latitude: {{  latitude  }}</p>
          <p>Longitude: {{  longitude  }}</p>
          <ion-button expand="block" @click="buscaCidade()">REGISTRAR LOCALIZAÇÃO</ion-button>
      </div>
      <div>
    <ion-card>
    <ion-card-header>
      <ion-card-title>Histórico</ion-card-title>
      <ion-card-subtitle>A lista abaixo apresenta o registro das localidades</ion-card-subtitle>
    </ion-card-header>

    <ion-card-content>
      <ion-col>
            <ul v-for="(cidade, index) in list" v-bind:key="index">
              <li>{{cidade}}</li>
            </ul>
          </ion-col>
              </ion-card-content>
  </ion-card>

      </div>
    </ion-content>
  </ion-page>
</template>

<script>
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonButton, IonCol, alertController, IonIcon } from '@ionic/vue';
import { defineComponent } from 'vue';
import { Geolocation } from '@capacitor/geolocation';
import { Http } from '@capacitor-community/http';
import { Storage } from '@ionic/storage'
import { ScreenBrightness } from '@capacitor-community/screen-brightness';

export default defineComponent({
  name: 'Home',
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonButton,
    IonCol,
    IonIcon,
  },

  data() {
    return {
      localStorage: new Storage(),
      latitude: 0,
      longitude: 0,
      coordenadas: '',
      cidade: '',
      list: []
    }
  },
  ionViewWillEnter(){
    this.printCurrentPosition();

  },
  created: async function () {
    await this.localStorage.create();

  },
  mounted: async function() {
    const list = await this.localStorage.get('list');
      if (JSON.parse(list) == null) {
        this.list = [];
      }
      else
        this.list = JSON.parse(list);
  },   

  methods: {
    printCurrentPosition: async function() {
      const coordinates = await Geolocation.getCurrentPosition();
      console.log('Current position: ', coordinates);

        this.latitude = coordinates.coords.latitude;
        this.longitude = coordinates.coords.longitude; 
    },
    buscaCidade: async function() {
        const ACCESS_KEY = '8cf68e16c982b92f6b8a0793785c61be';

        const options = {
          url: `http://api.positionstack.com/v1/reverse?access_key=${ACCESS_KEY}&query=${this.latitude},${this.longitude}&limit=1`
        };

        const response = await Http.get(options);
        console.log('Resposta recebida: ', response);

        this.cidade = response.data.data[0].locality + ',' + response.data.data[0].region_code;
          this.coordenadas = this.latitude + ' , ' + this.longitude;
          this.list.push("Localidade: " + this.cidade);
          await this.localStorage.set('list', JSON.stringify(this.list));
          console.log(this.list);
          const mensagem = "Você está em " + this.cidade;
          const alerta = await alertController
           .create({
          cssClass: 'my-custom-class',
          header: 'Localidade registrada!',
          message: mensagem,
          buttons: ['OK'],
        });
          await alerta.present();
    },
          async aumentarBrilho(){
    const fullBright = 1.0;
    ScreenBrightness.setBrightness({ fullBright });
          const msgalerta = "Você alterou o brilho da tela para o máximo.";
          const avis = await alertController
           .create({
          cssClass: 'my-custom-class2',
          header: 'Configuração aplicada',
          message: msgalerta,
          buttons: ['OK'],
        });
          await avis.present();
  }
  }
});
</script>