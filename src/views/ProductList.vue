<template>
  <div class="flex">
    <!-- Filter start -->
    <section class="text-gray-600 body-font w-4/12">
      <div class="container px-5 py-24 mx-auto">
        <h1 class="text-gray-700 title-font text-xl font-medium mt-4 mb-4">
          Manufacturers
        </h1>
        <form @onsubmit="event.preventDefault()">
          <div
            v-for="productListFilters in productData[0].manufacturers"
            :key="productListFilters.id"
          >
            <input
              type="checkbox"
              class="form-checkbox rounded text-pink-500"
              :id="productListFilters.name"
              :name="productListFilters.name"
              :value="productListFilters.name"
              v-model="manufacturersFilters"
            />
            <label :for="productListFilters.name" class="ml-2">{{
              productListFilters.name
            }}</label>
          </div>
        </form>
      </div>
    </section>
    <!-- Filter end -->

    <section class="text-gray-600 body-font w-8/12">
      <div class="container px-5 py-24 mx-auto">
        <!-- Selected filters start -->

        <div class="flex flex-wrap">
          <div class="w-full mb-4 mt-4 flex flex-col">
            <h1 class="text-gray-700 title-font text-xl font-medium">
              {{ productData[0].headline }}
            </h1>
            <p>Total products: {{ filteredProducts.length }}</p>
          </div>
          <div
            class="flex flex-row flex-nowrap w-full"
            v-if="manufacturerIsSelected"
          >
            <div
              v-for="(
                selectedFilters, selectedFiltersIdx
              ) in manufacturersFilters"
              :key="selectedFiltersIdx"
              class="flex"
              :class="manufacturersFilters.length === 1 ? 'w-full' : 'w-1/3'"
            >
              <a
                :value="selectedFilters"
                @click="deselectFilterValue(selectedFilters)"
                class="text-white cursor-pointer bg-indigo-500 border-0 py-2 px-6 focus:outline-none hover:bg-indigo-600 rounded text-lg mt-4 mb-4 inline-flex items-center"
              >
                {{ selectedFilters }}

                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  class="h-6 w-6 ml-2"
                  fill="none"
                  viewBox="0 0 24 24"
                  stroke="currentColor"
                  stroke-width="2"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    d="M6 18L18 6M6 6l12 12"
                  />
                </svg>
              </a>
            </div>
          </div>
          <!-- Selected filters end -->

          <!-- Products start -->
          <div
            class="lg:w-1/4 md:w-1/2 p-4 pl-0 pr-8 w-full flex flex-col"
            v-for="productListData in paginateFilteredProducts"
            :key="productListData.id"
          >
            <div class="bg-gray-100 p-6 rounded-lg w-full h-full">
              <img
                class="object-cover object-center"
                :width="productListData.image.width"
                :height="productListData.image.height"
                :alt="productListData.image.file"
                :title="productListData.image.file"
                :src="require(`../assets/img/${productListData.image.file}`)"
                v-if="productListData.image.file"
                loading="lazy"
              />
              <div class="mt-4">
                <h1
                  class="text-gray-900 title-font text-lg font-medium mb-2"
                  v-if="productListData.name"
                >
                  {{ productListData.name }}
                </h1>
                <h2
                  class="text-gray-700 title-font text-md font-medium mb-2"
                  v-if="productListData.id"
                >
                  ID: {{ productListData.id }}
                </h2>
                <h3
                  class="text-gray-500 text-xs tracking-widest title-font mb-2"
                  v-if="productListData.manufacturer"
                >
                  {{ productListData.manufacturer }}
                  <img
                    class="object-cover object-center w-full h-full block"
                    :width="productListData.manufacturerData.image.width"
                    :height="productListData.manufacturerData.image.height"
                    :alt="productListData.manufacturerData.image.file"
                    :title="productListData.manufacturerData.image.file"
                    :src="
                      require(`../assets/img/${productListData.manufacturerData.image.file}`)
                    "
                    v-if="productListData.manufacturerData.image.file"
                    loading="lazy"
                  />
                </h3>

                <p
                  class="mt-2"
                  v-if="
                    productListData.price.primary.raw ||
                    productListData.price.currencyCode
                  "
                >
                  {{ productListData.price.currencyCode }}
                  {{ productListData.price.primary.raw }}
                </p>
                <ul v-if="productListData.text.features" class="list-disc mt-4">
                  <li
                    v-for="(
                      productFeatures, productFeaturesIdx
                    ) in productListData.text.features"
                    :key="productFeaturesIdx"
                  >
                    {{ productFeatures }}
                  </li>
                </ul>
              </div>
            </div>
          </div>
          <!-- Products end -->

          <!-- Pagination start -->
          <div v-if="filteredProducts.length > 12" class="mt-4 flex w-full">
            <button
              class="flex mr-1"
              :disabled="pageNumber === 1 ? true : false"
              :class="pageNumber === 1 ? 'cursor-not-allowed' : ''"
              @click.prevent="setPage(pageNumber - 1)"
            >
              Prev
            </button>
            <button
              class="ml-1 mr-1"
              v-for="page in totalPages"
              :class="pageNumber === page ? 'text-blue-300 font-bold' : ''"
              @click.prevent="setPage(page)"
              :key="page"
            >
              {{ page }}
            </button>
            <button
              class="ml-1"
              @click.prevent="setPage(pageNumber + 1)"
              :class="pageNumber === totalPages ? 'cursor-not-allowed' : ''"
              :disabled="pageNumber === totalPages ? true : false"
            >
              Next
            </button>
          </div>
          <!-- Pagination end -->
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import Vue from 'vue';
import productDataJson from '../data/case/products.json';
import Component from 'vue-class-component';

// Including @component otherwise Vue class components doesnt work
@Component
export default class ProductList extends Vue {
  productData = productDataJson;

  manufacturersFilters = [];

  manufacturerIsSelected = false;

  defaultNumberOfProducts = 12;

  pageNumber = 1;

  //Computed start
  get filteredProducts() {
    if (this.manufacturersFilters.length !== 0) {
      this.manufacturerIsSelected = true;

      return this.productData[0].articles.filter((item) => {
        return (
          this.manufacturersFilters.length === 0 ||
          this.manufacturersFilters.includes(item.manufacturer)
        );
      });
    } else {
      this.manufacturerIsSelected = false;
    }
    return this.productData[0].articles;
  }

  get paginateFilteredProducts() {
    return this.filteredProducts.slice(
      (this.pageNumber - 1) * this.defaultNumberOfProducts,
      this.pageNumber * this.defaultNumberOfProducts
    );
  }

  get totalPages() {
    return Math.ceil(
      this.filteredProducts.length / this.defaultNumberOfProducts
    );
  }

  //Computed end

  //Methods start

  deselectFilterValue(element) {
    this.manufacturersFilters.splice(
      this.manufacturersFilters.indexOf(element),
      1
    );
  }

  setPage(idx) {
    if (idx <= 0 || idx > this.totalPages) {
      return;
    }
    this.pageNumber = idx;
  }

  //Methods end
}
</script>
