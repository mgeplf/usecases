<template>
   <div class="single-cell-insilico-experiments">
    <div class="title-uc">Please select a model</div>
    <div class="content-uc">
      <md-whiteframe md-elevation="1">
        <div class="search-container">
          <i class="material-icons">search</i>
          <input class="searchbox" type="text" v-model="filter" placeholder="Search e.g 'Hippocampus' or click on the item's title to filter">
        </div>
      </md-whiteframe>
      <models-list :models="models" v-on:selected="touched" v-on:tagfilter="addSearch"></models-list>
    </div>
   </div>
</template>

<script>
   import BspNmcModelList from '@/components/singlecellinsilicoexperiments/models-list.vue'
   const VIEWER_URL = 'https://bbp.epfl.ch/public/morph-view/'
   import collabAuthentication from '@/mixins/collabAuthentication.js'
   import createCollab from '@/mixins/createCollab.js'
   import modelsMixins from '@/mixins/models.js'
   
   export default {
      name: 'modelContainer',
      components: {
         'models-list': BspNmcModelList
      },
      data () {
        return {
          models: [],
          originalModels: [],
          filter: ''
        }
      },
      mixins: [collabAuthentication, createCollab],
      props: ['list_usecases', 'uc_name'],
      methods: {
        touched (modelItem) {
          let viewUrl = VIEWER_URL + modelItem.folderName + '.html'
          window.open(viewUrl, '_blank');
          var category = this.$route.path.split('/')[1]
          this.sendStatistics(null, this.uc_name, category, modelItem.folderName, null)
        },
        addSearch (obj) {
          this.filter += ' ' + obj.text
        },
        performNMC () {
          let fullList = modelsMixins.getNMCMetadata()
          this.models = this.models.concat(fullList)
          this.originalModels = this.models
        }
      },
      created () {
        let that = this
        document.querySelector('title').innerHTML = 'Models'
        this.models = modelsMixins.getBSPMetadata()
        if (window.requestIdleCallback) {
          window.requestIdleCallback(that.performNMC)
        } else {
          that.performNMC()
        }
      },
      watch: {
        'filter': async function (newVal) {
          this.models = await modelsMixins.search(newVal, this.originalModels)
        }
      }
   }
</script>
