<template>
  <div class="home">
    <section class="hero is-medium is-dark mb-6">
      <div class="hero-body has-text-centered">
        <p class="title mb-6">
          Welcome to Lawak Food and beverage
        </p>
        <p class="subtitle">
          Tempat asik untuk nongkrong 
        </p>
      </div>
    </section>

    <div class="columns-is-multiline">
        <div class="column is-12">
            <h2 class="is-size-2 has-text-centered">Latest products</h2>
        </div>
    </div>

    <ProductBox 
        v-for="product in latestProducts"
        v-bind:key="product.id"
        v-bind:product="product" />
    </div>
</template>

<script>
import axios from 'axios'

import ProductBox from '@/components/ProductBox.vue'

export default {
  name: 'Home',
  data(){
    return{
      latestProducts: []
    }
  },
  components: {
    ProductBox
  },
  mounted(){
    this.getLatestProducts()

    document.title = 'Home | Lawak FnB'
  },
  methods:{
  async getLatestProducts(){
    
    this.$store.commit('setIsLoading', true)

    await axios
        .get('/api/v1/latest-products/')
        .then(Response =>{
          this.latestProducts = Response.data
        })
        .catch(error =>{
          console.error();
        })

    this.$store.commit('setIsLoading', false)
    }
  }
}
</script>

