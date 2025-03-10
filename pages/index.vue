<template class="">
  <div style="background-color: #D3D3D3;
  min-height: 100vh;
  padding: 20px;">
    <div class="title text-center" style="margin-top: 3vh">
      <h1>Relatório de palestrantes</h1>
    </div>
    <v-card
      class="mx-auto"
      style="margin-top: 5vh; background-color: whitesmoke;"
      width="800"
      height="600"
      rounded="lg"
      elevation="6"
    >
      <v-row >
        <v-col>
          <v-form @submit.prevent>
              <v-row>
                <v-col class="d-flex justify-center">
                  <v-btn class="mt-8" color="primary" @click="generateCsv">
                    Gerar CSV
                  </v-btn>
                </v-col>
              </v-row>
              <v-row class="d-flex justify-center">
                <v-col cols="8">
                  <v-autocomplete class="mt-5" variant="outlined" multiple :items="dadosRequestSites.map(site => site.site)" v-model="dadosRequestSelecionados" />
                </v-col>
              </v-row>
              <v-row class="d-flex justify-center">
                <v-col>
                  <v-carousel
                    height="250"
                    v-if="dadosPalestrantes.length > 0"
                    color="primary"
                    :show-arrows="false"
                    hide-delimiter-background
                    progress="primary"
                  >
                    <v-carousel-item
                      v-for="(item, i) in dadosPalestrantes"
                      :key="i"
                      outlined
                    >
                      <v-sheet
                        height="60%"
                      >
                      <h1 class="text-center">{{ dadosRequestSelecionados[i] }}</h1>
                      <v-textarea class="ml-2" width="785" height="400" v-model="dadosPalestrantes[i]" outlined readonly>
                      </v-textarea>
                      </v-sheet>
                    </v-carousel-item>
                  </v-carousel>
                </v-col>   
              </v-row>
              <v-row class="d-flex justify-center">
                <v-btn
                  v-if="dadosPalestrantes.length > 0"
                  class="mt-5 pl-10 pr-10 ml-7"
                  color="green"
                  @click="downloadFile">
                  Baixar CSV
                </v-btn>
              </v-row>
          </v-form>
        </v-col>
      </v-row>
    </v-card>
  </div>
</template>
<script>
import Papa from 'papaparse';
import JSZip from 'jszip';
export default {
  data: () => {
    return {
      dadosPalestrantes: [],
      dadosRequestSites: [
        {
          id: 1,
          site: 'Abrella',
          url: 'https://api.brella.io/api/public/events/bitcoin2025/speakers?page[size]=500',
          attributeName: 'first-name',
          attributeSurname: 'last-name',
          attributeJob: 'job-title',
          attributeCompany: 'company-name',
        },
        {
          id: 2,
          site: 'MiningDisrupt',
          url: 'https://miningdisrupt.com/#speakers',
        },
      ],
      dadosRequestSelecionados: [],
    };
  },

  methods: {
    async scrape(site) {
      const apiKey = "1dec614eb34c247278ca3b83ee59c15e";
      const targetUrl = encodeURIComponent(`${site.url}`);
      const apiUrl = `https://api.scraperapi.com?api_key=${apiKey}&url=${targetUrl}`;
      const parser = new DOMParser();

      try {
        const response = await fetch(apiUrl);
        const text = await response.text();
        const doc = parser.parseFromString(text, "text/html");
        let dadosIteracao = [];
        if (site.site === 'MiningDisrupt') {
          const galleryDiv = doc.querySelector("#gallery08");
          
          if (!galleryDiv) {
            console.error("Gallery div not found");
            return;
          }
          const liElements = galleryDiv.querySelectorAll("div ul li");
  
          const extractedData = Array.from(liElements).map(li => {
            const imgElement = li.querySelector("a span img");
            return imgElement ? imgElement.alt : "No alt found";
          });
          extractedData.forEach((palestrante) => {
            const nomePalestrante = palestrante.split(' - ');
            dadosIteracao.push({ 
              name: nomePalestrante[0],
              title: nomePalestrante[1],
              company: nomePalestrante[2],
            });
          });
          this.dadosPalestrantes.push(Papa.unparse(dadosIteracao));
        }
        console.log(this.dadosPalestrantes);
      } catch (error) {
        console.error("Error:", error);
      }
    },

    async generateCsv() {
      try {
        const selectedSites = this.dadosRequestSelecionados.map(
          (siteName) => this.dadosRequestSites.find((site) => site.site === siteName)
        ).filter(Boolean);
        
        selectedSites.forEach(async (site) => {
          if (site.site === 'MiningDisrupt') {
            this.scrape(site);
            return;
          }
          const response = await fetch(site.url);
          const items = await response.json();
          let dadosIteracao = [];
          items.data.forEach(palestrante => {
            const palestranteNome = palestrante.attributes[site.attributeName] + ' ' + palestrante.attributes[site.attributeSurname];
            dadosIteracao.push({ 
              name: palestranteNome, 
              title: palestrante.attributes[site.attributeJob], 
              company: palestrante.attributes[site.attributeCompany],
            });
          });
          this.dadosPalestrantes.push(Papa.unparse(dadosIteracao));
        });
      } catch (error) {
        console.log(error);
      }
    },
    async downloadFile() {
      const zip = new JSZip();
      
      this.dadosPalestrantes.forEach((dados, index) => {
        zip.file(`palestrantes-${this.dadosRequestSelecionados[index]}.csv`, dados);
      });

      const zipBlob = await zip.generateAsync({ type: "blob" });
      
      const link = document.createElement('a');

      const url = URL.createObjectURL(zipBlob);
      
      link.href = url;
      link.download = 'representantes.zip';
      
      link.click();
      
      URL.revokeObjectURL(url);
    },
  },
};
</script>