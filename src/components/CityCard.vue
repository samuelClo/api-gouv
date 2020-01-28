<template>
      <ion-card>
<ion-card-header>
  <ion-card-subtitle>{{ headerText }}</ion-card-subtitle>
  <ion-card-title>{{ bodyText }}</ion-card-title>
</ion-card-header>
<ion-card-content>
  {{footerText}}
</ion-card-content>
      </ion-card>
</template>

<script>
export default {
  name: 'CityCard',
  props: {
    headerText: {
        required: true,
    },
    bodyText: {
        required: false,
    },
    footerText: {
        required: true,
    }
  }
}
</script>