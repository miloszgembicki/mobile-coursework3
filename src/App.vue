<template>
  <div id="app">

    <header>

      <header class="navigation">

        <div>
          <h1 v-text="sitename"></h1>
        </div>

        <div style="align-self: center;">
          <button @click="showCheckout" v-if="cartCountItems > 0 || this.showProduct == false">
            {{ cartCountItems }}
            Cart
            <font-awesome-icon icon="fas fa-shopping-cart" />
          </button>
          <button id="disabled_button" disabled="disabled" v-else>
            Cart
            <font-awesome-icon icon="fas fa-shopping-cart" /> </button>
        </div>

        <div style="align-self: center;">
          <button v-if="testConsole" @click="toggleShowTestConsole">
            Test Console
            <font-awesome-icon icon="fas fa-text-height" />
          </button>
        </div>

      </header>

      <div style="text-align: center;" v-if="testConsole && showTestConsole" class="test-console">
        <button @click="saveProductToDB" class="test-elem">
          Test Save a Product to the DB
          <font-awesome-icon icon="fas fa-save" />
        </button>
        <button @click="deleteAllCaches" class="test-elem">
          Delete All Caches
          <font-awesome-icon icon="fas fa-trash" />
        </button>
        <button @click="reloadPage" class="test-elem">
          Reload Page
          <font-awesome-icon icon="fas fa-sync" />
        </button>
        <strong class="test-elem">HTTPS Test: </strong><a v-bind:href="serverURL" target="_blank">link</a>
        <button @click="unregisterAllServiceWorkers" class="test-elem">
          Unregister All Service Workers
          <font-awesome-icon icon="fab fa-uniregistry" />
        </button>
      </div>

    </header>
    <main>

      <component :is="currentView" :sortProducts="sortProducts" :cart="cart" @add-item-to-cart="addToCart"
        @manage-remove-item="removeFromCart"></component>

    </main>

  </div>
</template>

<script>
import ProductList from "./components/ProductList.vue";
import Checkout from "./components/Checkout.vue";

export default {
  name: "App",
  data() {
    return {
      sitename: "Classes Shop",
      cart: [],
      currentView: ProductList,
      products: [],
      testConsole: true,
      showTestConsole: false,
      serverURL: "https://coursework-env.eba-8ryb6pbz.eu-west-2.elasticbeanstalk.com/collections/products",
      showProduct: true,
    }
  },
  components: { ProductList, Checkout },
  created: function () {

    if ("serviceWorker" in navigator) {
      navigator.serviceWorker.register("service-worker.js");
    }

    let app = this;

    fetch(this.serverURL).then(
      function (response) {
        response.json().then(function (json) { app.products = json; })
      }
    );
  },
  methods: {
    toggleShowTestConsole() {
      this.showTestConsole = !this.showTestConsole;
    },
    saveProductToDB() {
      const newProduct = {
        "id": 1011,
        "imgALT": "Music img",
        "imgSRC": "images/music.png",
        "title": "Music Test",
        "location": "Manchester",
        "price": 12.49,
        "spaces": 10,
        "spacesLeft": 10
      }

      let app = this;

      //set the url to your server and route
      fetch(this.serverURL, {
        method: "POST", //set the HTTP method as "POST"
        headers: {
          "Content-Type": "application/json", //set the data type as JSON
        },
        body: JSON.stringify(newProduct) //need to stringify the JSON object
      }).then(
        function (response) {
          response.json().then(
            function (json) {
              console.log("Success: " + json.acknowledged);

              app.products.push(newProduct);
            }
          )
        }
      );
    },
    deleteAllCaches() {
      caches.keys().then(function (names) {
        for (let name of names)
          caches.delete(name);
      });
      console.log("All caches deleted!");
    },
    reloadPage() {
      window.location.reload();
    },
    unregisterAllServiceWorkers() {
      navigator.serviceWorker.getRegistrations().then(function (registrations) {
        for (let registration of registrations) {
          registration.unregister();
        }
      });
      console.log("Service Workers Unregistered!");
    },
    showCheckout() {
      if (this.currentView === ProductList) { this.currentView = Checkout } else { this.currentView = ProductList }
    },
    addToCart: function (product) {
      this.cart.push(product);
      product.spacesLeft -= 1;
    },
    removeIndexCart: function (index) {
      this.cart.splice(index, 1);
    },
    removeFromCart: function (product) {
      let index = this.cart.findIndex(item => item.id === product.id);
      if (index !== -1) {
        this.removeIndexCart(index);
      }
      if (!this.cartCountItems) {
        this.currentView = ProductList;
      }
    },
  },
  computed: {
    sortProducts() {
      function compare(a, b) {
        if (a.price > b.price) return +1;
        if (a.price < b.price) return -1;
        return 0;
      }
      return this.products.sort(compare);
    },
    cartCountItems: function () {
      return this.cart.length || "";
    },
  },
}  
</script>