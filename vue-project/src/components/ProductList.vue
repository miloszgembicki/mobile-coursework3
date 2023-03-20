<template>
   <div id="app">

      <div class="class_windows" v-for="product in sortProducts">
         <div id="panel_layout">
            <div id="text_layout">

               <img id="img" v-bind:src="product.imgSRC" v-bind:alt="product.imgALT" />
               <p>Subject: {{ product.title }}</p>
               <p>Location: {{ product.location }}</p>
               <p>Price: £{{ product.price }}</p>
               <p>Assigned Spaces: {{ product.spaces }} </p>

               <div id="button_align">
                  <span v-if="itemsLeft(product) === 0">
                     No spaces available!
                  </span>

                  <span v-else-if="itemsLeft(product) < 5">
                     Only {{ itemsLeft(product) }} left!
                  </span>

                  <span v-else>
                     Buy Now!
                  </span>

                  <button v-on:click="addToCart(product)" v-if="canAddToCart(product)">Add to
                     cart</button>
                  <button id="disabled_button" disabled="disabled" v-else>Add to cart</button>
               </div>


               <!-- <div id="panel_layout">
         <div id="text_layout">

            <img id="img" v-bind:src="product.imgSRC" v-bind:alt="product.imgALT" />
            <p>Subject: {{ product.title }}</p>
            <p>Location: {{ product.location }}</p>
            <p>Price: £{{ product.price }}</p>
            <p>Assigned Spaces: {{ product.spaces }} </p>

            <div id="button_align">
               <span v-if="itemsLeft(product) === 0">
                  No spaces available!
               </span>

               <span v-else-if="itemsLeft(product) < 5">
                  Only {{ itemsLeft(product) }} left!
               </span>

               <span v-else>
                  Buy Now!
               </span>

               <button v-on:click="addToCart(product)" v-if="canAddToCart(product)">Add to
                  cart</button>
               <button id="disabled_button" disabled="disabled" v-else>Add to cart</button>
            </div>

         </div>
      </div> -->
            </div>
         </div>
      </div>
   </div>
</template>

<script>
export default {
   name: "ProductList",
   props: ["sortProducts", "cart"],
   data() {
      return {}
   },
   methods: {
      canAddToCart: function (product) {
         return product.spaces > this.cartCount(product);
      },
      cartCount: function (id) {
         let count = 0;
         for (let i = 0; i < this.cart.length; i++) {
            if (this.cart[i] === id) {
               count++;
            }
         }
         return count;
      },
      addToCart: function (product) {
         //this.cart.push(product);
         this.$emit("add-item-to-cart", product);
      },
      itemsLeft: function (product) {
         return product.spaces - this.cartCount(product);
      },
   }
}
</script>