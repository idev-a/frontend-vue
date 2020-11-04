<template>
  <v-container
    id="dashboard"
    fluid
    tag="section"
  >
    <v-card
      icon="mdi-account-multiple"
      title="Dashboard"
      class="px-5 py-3"
    >
      <v-card-title>
        Dashboard
      </v-card-title>
      <v-card-text>
        <div class="d-flex justfify-around">
          <v-tooltip bottom>
            <template v-slot:activator="{ on }">
              <v-btn class="primary mt-3" v-on="on" :disabled="!is_new" @click="run_sysco_scraper">
                Run Scraper
                <v-icon right>mdi-send</v-icon>
              </v-btn>
            </template>
            <span>Run the scraper in the background</span>
          </v-tooltip>

          <v-tooltip bottom>
            <template v-slot:activator="{ on }">
              <v-btn class="success mt-3" v-on="on" @click="readData">
                Read 
                <v-icon right>mdi-database</v-icon>
              </v-btn>
            </template>
            <span>Read the Item List as Table</span>
          </v-tooltip>
          <v-file-input
            ref="myfile" 
            v-model="file"
            accept=".xlsx"
            placeholder="Import Item List"
            label="Datasheet"
            :loading="loading"
          ></v-file-input>
          <v-tooltip bottom>
            <template v-slot:activator="{ on }">
              <v-btn class="success mt-3 ml-2" v-on="on" :disabled="!file"  @click="uploadSheet">
                <v-icon>mdi-upload</v-icon>
              </v-btn>
            </template>
            <span>Upload New Item List</span>
          </v-tooltip>
          <v-tooltip bottom>
            <template v-slot:activator="{ on }">
              <v-btn class="success mt-3" v-on="on" @click="downloadFile">
                <v-icon>mdi-download</v-icon>
              </v-btn>
            </template>
            <span>Download Result</span>
          </v-tooltip>
        </div>
        <div v-if="!is_new" class="mt-3">
          <div>{{message}}</div>
          <div class="mt-2">If you want to run the scraper again, please refresh the page.</div>
        </div>
      </v-card-text>
      <v-card-title>
        <v-text-field
          v-model="search"
          append-icon="mdi-magnify"
          label="Search"
          clearable
          class="mb-5"
          single-line
          hide-details
        ></v-text-field>
        <v-spacer></v-spacer>
      </v-card-title>
      <v-card-text>
        <v-data-table
          :loading="loading"
          :headers="tableHeaders"
          :items="items"
          fixed-header
          item-key="id"
          :search="search"
        > 
        </v-data-table>
      </v-card-text>
    </v-card>

    <v-snackbar
      v-model="snackbar"
      bottom
      :color="color"
      >
      {{ message }}
      <template v-slot:action="{ attrs }">
        <v-btn
          dark
          text
          v-bind="attrs"
          @click="snackbar = false"
        >
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </v-container>
</template>

<script>
  import { Get, Post, BASE_API } from '@/api'
  import upperFirst from 'lodash/upperFirst'

  export default {
    name: 'DashboardDashboard',

    data () {
      return {
        loading: false,
        snackbar: false,
        message: '',
        color: 'success',
        is_new: true,
        search: '',
        items: [],
        headers: [],
        file: null,
        rules: {
          required: value => {
            return !!value || 'This field is required.'
          },
        }
      }
    },

    mounted () {
    },

    computed: {
      page() {
        return Number(localStorage.getItem('page')) || 5
      },
      tableHeaders () {
        return this.headers.map(header => {
          return {
            text: header,
            value: upperFirst(header)
          }
        })
      }
    },

    methods: {
      async run_sysco_scraper() {
        try {
          const res = await Get('admin/sysco')
          this.message = 'Successfully run the scraper, you will get notificed via email about the completion soon.'
        } catch (e) {
          this.color = 'default'
          this.message = 'Something went wrong. Please contact the admin.'
        }
        this.snackbar = true
        this.is_new = false
      },
      downloadFile () {
        var link = document.createElement("a");
        link.setAttribute("href", `${BASE_API}/api/static/download`);
        link.setAttribute("download", 'Item List.xlsx');
        link.style.visibility = 'hidden';
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
      },
      async uploadSheet () {
        let formData = new FormData()
        formData.append("file", this.file, this.file.name)
        try {
          const res = await Post('admin/upload', formData)
          this.message = res.message
          this.color = res.status
        } catch(e) {
          this.color = 'error'
          this.message = 'Something wrong happened on the server.'
        } finally {
          this.loading = false
          this.snackbar = true
          this.file = null
        }
      },
      async readData () {
        this.loading = true
        const res = await Get('admin/read')
        this.snackbar = true
        this.color = res.status
        this.message = res.message
        this.items = res.items
        this.headers = res.headers
        this.loading = false
      }
    },
  }
</script>

<style>
</style>