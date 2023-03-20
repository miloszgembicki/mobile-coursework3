<template>
   <div id="app">
      <div class="welcome_page">

         <h1 v-if="!cart.length">Your cart is EMPTY, click "Cart" to shop for classes.</h1>


         <div v-if="cart.length">

            <h1>Shopping Cart</h1>

            <div class="class_windows" v-for="product in cart">
               <div id="panel_layout">
                  <div id="text_layout">

                     <img id="img" v-bind:src="product.imgSRC" v-bind:alt="product.imgALT" />
                     <p>Subject: {{ product.title }}</p>
                     <p>Location: {{ product.location }}</p>
                     <p>Price: £{{ product.price }}</p>

                     <div id="button_align">
                        <button v-on:click="removeFromCart(product)">Remove from
                           cart</button>
                     </div>

                  </div>
               </div>
            </div>

            <h1>Checkout</h1>

            <div id="checkout_form">
               Name:
               <input type="text" @keydown="checkName($event)" v-model="checkout.name">
               <span v-html="errorName"></span>

               Phone:
               <input type="text" @keydown="checkPhoneNumber($event)" v-model.trim="checkout.phoneNumber">
               <span v-html="errorPhone"></span>

               <button v-on:click="saveCartToDB" v-if="goodPhone && goodName">Place Order</button>
               <button id="disabled_button" disabled="disabled" v-else>Place Order</button>
            </div>

         </div>
      </div>
   </div>
</template>

<script>
export default {
   name: "Checkout",
   props: ["cart", "currentView"],
   data() {
      return {
         goodName: false,
         goodPhone: false,
         errorName: "",
         errorPhone: "",
         checkout: {
            name: "",
            phoneNumber: "",
         },
      }
   },
   methods: {
      removeFromCart: function (product) {
         this.$emit("manage-remove-item", product);
         //this.cart.splice(this.cart.indexOf(product.id));
      },
      checkName: function (e) {
         let keyCode = e.keyCode || e.which;
         let char = String.fromCharCode(keyCode);
         if (/^[A-Za-z]+$/.test(char) || keyCode == 8 || keyCode == 9 || keyCode == 13 || keyCode == 27) {
            this.goodName = true;
            this.errorName = "";
         } else {
            this.errorName = "LETTERS ONLY!";
            this.goodName = false;
         }
      },
      checkPhoneNumber: function (e) {
         let keyCode = e.keyCode || e.which;
         let char = String.fromCharCode(keyCode);
         if (/^[0-9]*$/.test(char) || keyCode == 8 || keyCode == 9 || keyCode == 13 || keyCode == 27) {
            this.goodPhone = true;
            this.errorPhone = "";
         } else {
            this.errorPhone = "NUMBERS ONLY!";
            this.goodPhone = false;
         }
      },
      saveCartToDB: function () {
         let app = this;
         let productIds = [];
         let productCounts = {};
         for (let i = 0; i < this.cart.length; i++) {
            let id = this.cart[i].id;
            if (productCounts[id]) {
               productCounts[id]++;
            } else {
               productCounts[id] = 1;
            }
         }

         for (const [lesson_id, number_of_spaces] of Object.entries(productCounts)) {
            productIds.push({ lesson_id, number_of_spaces });
         }

         for (let i = 0; i < this.cart.length; i++) {
            let id = this.cart[i].id;
            let objectId = this.cart[i]._id;

            fetch("https://coursework-env.eba-8ryb6pbz.eu-west-2.elasticbeanstalk.com/collections/products/" + objectId, {
               method: "PUT",
               headers: {
                  "Content-Type": "application/json",
               },
               body: JSON.stringify({
                  "spaces": this.cart[i].spaces - productCounts[id],
                  "spacesLeft": this.cart[i].spaces - productCounts[id]
               })
            }).then(
               function (response) {
                  response.json().then(
                     function (json) {
                        console.log("Success: " + json.acknowledged);
                     }
                  )
               }
            )
         }

         const newProduct = {
            "name": this.checkout.name,
            "phone_number": this.checkout.phoneNumber,
            "cart": productIds,
         }

         fetch("https://coursework-env.eba-8ryb6pbz.eu-west-2.elasticbeanstalk.com/collections/cart", {
            method: "POST",
            headers: {
               "Content-Type": "application/json",
            },
            body: JSON.stringify(newProduct)
         }).then(
            function (response) {
               response.json().then(
                  function (json) {
                     alert("Success: " + json.acknowledged);
                     console.log("Success: " + json.acknowledged);
                     app.products.push(newProduct);
                     this.currentView = ProductList;
                  }
               )
            }
         )
      }
   }
}
</script>