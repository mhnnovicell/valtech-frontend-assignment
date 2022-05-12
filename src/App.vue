<template>
  <div id="app" class="container px-4 mx-4">
    <ProductList></ProductList>
  </div>
</template>

<script>
import Vue from 'vue';
import Component from 'vue-class-component';
import ProductList from './views/ProductList';

// HelloWorld class will be a Vue component
@Component({
  // Specify `components` option.
  // See Vue.js docs for all available options:
  // https://vuejs.org/v2/api/#Options-Data
  components: {
    ProductList,
  },
})
export default class App extends Vue {}
</script>
