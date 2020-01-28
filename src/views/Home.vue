<template>
  <div class="ion-page">
    <ion-header>
      <ion-toolbar>
        <ion-title>SearchEngine</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content class="ion-padding">
      <h1>Votre commune : </h1>
      <ion-item class="search">
        <ion-label position="floating">Enter your zip-code</ion-label>
        <ion-input
          :value="text"
          @input="text = $event.target.value"
          v-on:input="this.handleChange"
        >
        </ion-input>
      </ion-item>

      <span class="errorText" v-if="this.errorText">Le code postal est pas dans le bon format</span>
      <ul id="suggestionBox" v-if="suggestionContent.length && this.researchType === 'city' && text">
        <li v-for="suggestion in suggestionContent" @click="selectSuggestion(suggestion.code)">
          {{ suggestion.nom }}
        </li>
      </ul>
      <ion-button id="submitButton" @click="handleClick()" expand="block">Rechercher</ion-button>
      <router-link :to="{ path: 'about' }">About</router-link>

      <CityCard
        v-if="dataReceive"
        v-for="item in dataReceive"
        :headerText="item.headerText"
        :bodyText="item.bodyText"
        :footerText="item.footerText"
      />
    </ion-content>
  </div>
</template>

<script>
    import CityCard from '../components/CityCard.vue'

    export default {
        name: "home",
        components: {
            CityCard,
        },
        data: function () {
            return {
                text: '',
                errorText: false,
                dataReceive: [],
                researchType: '',
                suggestionContent: [],
            }
        },
        methods: {
            selectSuggestion(text) {
                this.text = text
                this.handleClick()
            },
            handleChange() {
                const searchText = this.text.trim()
                const regExpCity = /^[A-Za-z]+$/

                if (!searchText.match(regExpCity))
                    return

                this.handleClick()
            },
            handleClick() {
                if (this.checkReasearchType())
                    return
                this.errorText = false
                this.fetchData()
            },
            fetchData() {
                if (this.researchType === 'city') {
                    fetch(this.RESEARCH_DATA[this.researchType], {})
                        .then(res => res.json())
                        .then(data => {
                             if (data)
                                 this.suggestionContent = data
                        })
                        .catch(() => {
                            this.fetchDataAlert()
                        })
                } else {
                    fetch(this.RESEARCH_DATA[this.researchType], {})
                        .then(res => res.json())
                        .then(data => {
                            this.calculateData(data)
                            this.text = ''
                        })
                        .catch(data => {
                            console.error(data)
                            this.fetchDataAlert()
                        })
                }
            },
            fetchDataAlert() {
                return this.$ionic.alertController
                    .create({
                        header: 'Erreur',
                        subHeader: 'Impossible de trouver la ressource demander',
                        message: 'Veuillez vérifier votre connexion internet et contacter nous par mail si l\'erreur persiste',
                        buttons: ['OK'],
                    })
                    .then(a => a.present())
            },
            checkReasearchType() {
                const searchText = this.text.trim()
                const regExpPostalCode = /^(?:[0-8]\d|9[0-8])\d{3}$/
                const regExpDepartement = /^[0-9]{2}$/
                const regExpCity = /^[A-Za-z]+$/

                if (searchText.match(regExpPostalCode))
                    this.researchType = 'postalCode'
                else if (searchText.match(regExpDepartement))
                    this.researchType = 'departement'
                else if (searchText.match(regExpCity))
                    this.researchType = 'city'
                else {
                    this.errorText = true
                    return true
                }
            },
            calculateData(data) {
                const searchText = this.researchType

                if (searchText === 'postalCode') {
                    this.dataReceive = data.map(el => {
                        return {
                            headerText: el.nom,
                            bodyText: `Habitants : ${el.population}`,
                            footerText: `Département : ${el.codeRegion}`,
                        }
                    })
                } else if (searchText === 'departement') {
                    this.dataReceive = data.map(el => {
                        return {
                            headerText: el.nom,
                            bodyText: el.codePostaux,
                            footerText: el.population,
                        }
                    })
                } // } else if (searchText === 'departement') {
                //     this.dataReceive = data.map(el => {
                //         return {
                //             headerText: el.nom,
                //             bodyText: el.codeRegion,
                //             footerText: el.population,
                //         }
                //     })
                // }
            }
        },
        computed: {
            RESEARCH_DATA() {
                return {
                    postalCode: `https://geo.api.gouv.fr/communes?codePostal=${this.text}`,
                    departement: `https://geo.api.gouv.fr/departements/${this.text}/communes`,
                    city: `https://geo.api.gouv.fr/departements?nom=${this.text}&fields=nom,code,codeRegion`,
                }
            },
        }
    };
</script>

<style lang="scss">
  #submitButton {
    margin-top: 10px;
  }

  #suggestionBox {
    list-style: none;
    padding: 10px 0;
    width: 100%;
    max-height: 115px;
    overflow-y: scroll;
    background-color: #ffff;
    box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24);
    margin-top: 10px;
    li {
      cursor: pointer;
      margin-bottom: 13px;
    }
  }

  .errorText {
    color: #FF676D;
  }
</style>