<template class="">
  <div style="background-color: #D3D3D3;
  min-height: 100vh;
  padding: 20px;">
    <div class="title text-center" style="margin-top: 15vh">
      <h1>Relatório de palestrantes</h1>
    </div>
    <v-card
      class="mx-auto"
      style="margin-top: 5vh; background-color: lightgray;"
      width="800"
      height="600"
      rounded="lg"
      elevation="6"
    >
      <v-row >
        <v-col>
          <v-form @submit.prevent>
            <v-row >
              <v-col class="d-flex justify-center">
                <v-btn class="mt-15 pl-10 pr-10 ml-7" color="primary" @click="generateCsv">
                  Gerar CSV
                </v-btn>
              </v-col>
              <v-col class="d-flex justify-center">      
                <v-textarea class="mt-15 ml-4 mr-4" width="800" height="400" v-model="dadosPalestrantes" outlined readonly>
                  {{ dadosPalestrantes }}
                </v-textarea>
              </v-col>
              <v-col class="d-flex justify-center">
                <v-btn
                  v-if="dadosPalestrantes != ''"
                  class="mt-5 pl-10 pr-10 ml-7"
                  color="green"
                  @click="downloadFile">
                  Baixar CSV
                </v-btn>
              </v-col>
            </v-row>
          </v-form>
        </v-col>
      </v-row>
    </v-card>
  </div>
</template>
<script>
import Papa from 'papaparse';
export default {
  data: () => {
    return {
      dadosPalestrantes: [],
    };
  },

  methods: {
    async generateCsv() {
      try {
        const response = await fetch('https://api.brella.io/api/public/events/bitcoin2025/speakers?page[size]=500');
        const items = await response.json();
        items.data.forEach(palestrante => {
          const palestranteNome = palestrante.attributes['first-name'] + ' ' + palestrante.attributes['last-name'];
          this.dadosPalestrantes.push({ 
            name: palestranteNome, 
            title: palestrante.attributes['job-title'], 
            company: palestrante.attributes['company-name'],
          });
        });
        this.dadosPalestrantes = Papa.unparse(this.dadosPalestrantes);
      } catch (error) {
        console.log(error);
      }
    },
    downloadFile() {
      const blob = new Blob([this.dadosPalestrantes], { type: 'text/plain' });
      
      const link = document.createElement('a');
    
      const url = URL.createObjectURL(blob);
      
      link.href = url;
      link.download = 'representantes.txt';
      
      link.click();
      
      URL.revokeObjectURL(url);
    },
  },
};
</script>