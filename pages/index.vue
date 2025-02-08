<template>
  <v-app>
    <v-container>
      <!-- MAIN PRODUCTS Header -->
      <v-row class="mb-4">
        <v-col cols="12">
          <h2 class="header-title">MAIN PRODUCTS</h2>
        </v-col>
      </v-row>

      <!-- Oval Button Styled List -->
      <v-row class="mb-4 justify-center">
        <v-col cols="12" class="text-center">
          <v-btn v-for="item in itemList" :key="item.id" class="mr-3 mb-2 oval-button" @click="handleItemClick(item)">
            {{ item.ProductType }}
          </v-btn>
        </v-col>
      </v-row>

      <!-- Product Grid -->
      <v-row>
        <v-col v-for="product in products" :key="product.id" cols="12" md="4">
          <v-card class="hover-card" @click="goToProduct(product.id)">
            <v-img :src="product.image" height="200px"></v-img>

            <v-card-title class="d-flex justify-space-between align-center">
              <span>{{ product.name }}</span>
              <v-btn icon class="cart-icon" @click.stop="addToCart(product, quantity)">
                <v-icon style="color: #FFA900;">mdi-cart-minus</v-icon>
              </v-btn>
            </v-card-title>

            <v-card-subtitle style="color: black;">₱{{ product.price }}</v-card-subtitle>

            <!-- Sold Quantity Display -->
            <v-card-subtitle class="sold-info" style="color: black;">Sold: {{ product.soldQuantity }}</v-card-subtitle>

            <!-- See More and Buy Now buttons -->
            <v-card-actions class="d-flex justify-space-between">
              <span class="see-more" @click.stop="goToProduct(product.id)">See More..</span>
              <v-btn class="buy-now-btn" @click.stop="buyNow(product, quantity)" :disabled="loading"
                style="background-color: #FFA900; color: white;">
                <template v-if="loading">
                  <v-progress-circular indeterminate size="24" color="white" />
                </template>
                <template v-else>
                  Buy Now
                </template>
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>

      <!-- SEE MORE PRODUCTS Button -->
      <v-row class="mt-4 justify-center">
        <v-col cols="12" class="text-center">
          <v-btn class="see-more-btn" @click="goToGallery" style="background-color: #FFA900;">
            SEE MORE PRODUCTS <v-icon right>mdi-arrow-right</v-icon>
          </v-btn>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script>
import { firestore, auth } from '~/plugins/firebase';
import { collection, getDocs, addDoc, query, where, updateDoc, onSnapshot } from 'firebase/firestore'; // Import query and where
import { getAuth } from 'firebase/auth'; // Import Firebase Authentication

export default {
  data() {
    return {
      products: [],
      itemList: [],
      quantity: 1,
      loading: false,  // Add a loading state
    };
  },
  async created() {
    try {
      // Real-time listener for Categories
      onSnapshot(collection(firestore, 'Categories'), (categoriesSnapshot) => {
        this.itemList = categoriesSnapshot.docs.map(doc => ({
          id: doc.id,
          ProductType: doc.data().ProductType,
        }));
      });

      // Real-time listener for Products
      onSnapshot(collection(firestore, 'Products'), async (productsSnapshot) => {
        const productDocs = productsSnapshot.docs;

        // Fetch orders in real-time
        onSnapshot(collection(firestore, 'Orders'), (ordersSnapshot) => {
          const orders = ordersSnapshot.docs.map(doc => doc.data());

          // Map products and calculate sold quantities
          this.products = productDocs.map(doc => {
            const productId = doc.id;

            // Calculate total sold quantity for this product
            const totalSold = orders
              .filter(order => order.cartItems.some(item => item.productID === productId))
              .reduce((sum, order) => {
                const productInOrder = order.cartItems.find(item => item.productID === productId);
                return sum + (productInOrder ? productInOrder.Quantity : 0);
              }, 0);

            return {
              id: productId,
              name: doc.data().ProductName,
              price: doc.data().Price,
              image: doc.data().Image,
              soldQuantity: totalSold,
            };
          });
        });
      });

    } catch (error) {
      console.error("Error fetching data:", error);
    }
  },
  methods: {
    handleItemClick(item) {
      this.$router.push({ path: '/gallery', query: { category: item.ProductType } });
    },
    async addToCart(product, quantity) {
      try {
        const user = auth.currentUser; // Get the current logged-in user

        if (!user) {
          // Redirect to sign-in page if the user is not authenticated
          this.$router.push('/sign/signin');
          return;
        }

        // Proceed if the user is authenticated
        const cartRef = collection(firestore, 'Cart'); // Reference to the Cart collection

        // Create a query to check if the product already exists in the user's cart and is not confirmed
        const productQuery = query(
          cartRef,
          where('userID', '==', user.uid),
          where('ProductID', '==', product.id),
          where('orderStatus', '==', 'Not Confirmed') // Exclude confirmed items
        );

        const querySnapshot = await getDocs(productQuery); // Get query snapshot

        if (querySnapshot.empty) {
          // If no existing product, add the product to the cart
          await addDoc(cartRef, {
            userID: user.uid, // Add user ID
            ProductID: product.id, // Add Product ID
            Quantity: quantity, // Add Quantity
            orderStatus: "Not Confirmed" // Set initial status
          });

          console.log('Product added to cart:', product.name); // Log success message
        } else {
          // If the product already exists in the cart, update the quantity
          const cartDoc = querySnapshot.docs[0]; // Get the first matching document
          const updatedQuantity = cartDoc.data().Quantity + quantity; // Update the quantity

          await updateDoc(cartDoc.ref, {
            Quantity: updatedQuantity, // Update the quantity field
          });

          console.log('Cart updated with new quantity:', updatedQuantity); // Log success message
        }

        // Update the cart count
        this.cartCount += quantity;
      } catch (error) {
        console.error('Error adding or updating product in cart:', error); // Log any errors during adding or updating
      }
    },
    // async buyNow(product, quantity) {
    // try {
    //   const user = auth.currentUser;

    //   if (!user) {
    //     // Redirect to sign-in if the user is not authenticated
    //     this.$router.push('/sign/signin');
    //     return;
    //   }

    //   // Create a temporary cart item to pass to the checkout page
    //   const cartItem = {
    //     userID: user.uid,
    //     ProductID: product.id,
    //     productName: product.ProductName,
    //     price: product.price,
    //     Quantity: quantity,
    //     image: product.image || this.defaultImage,
    //   };

    //   // Redirect to checkout page with cart item as query
    //   this.$router.push({
    //     path: '/checkout',
    //     query: {
    //       items: JSON.stringify([cartItem]), // Convert the cart item to a query string
    //     },
    //   });
    // } catch (error) {
    //   console.error('Error processing Buy Now action:', error);
    // }
    // },
    async buyNow(product, quantity) {
    try {
      const user = auth.currentUser;

      if (!user) {
        // Redirect to sign-in if the user is not authenticated
        this.$router.push('/sign/signin');
        return;
      }

      // Create a temporary cart item to pass to the checkout page
      const cartItem = {
        userID: user.uid,
        ProductID: product.id,
        productName: product.name, // Corrected field name
        price: product.price,
        Quantity: quantity,
        image: product.image || this.defaultImage,
      };

      // Redirect to checkout page with cart item as query
      this.$router.push({
        path: '/checkout',
        query: {
          items: JSON.stringify([cartItem]), // Convert the cart item to a query string
        },
      });
    } catch (error) {
      console.error('Error processing Buy Now action:', error);
    }
},
    goToProduct(productId) {
      this.$router.push(`/product/${productId}`);
    },
    goToGallery() {
      this.$router.push(`/gallery`);
    },
    // buyNow(product) {
    //   this.goToProduct(product.id);
    // },
  },
};
</script>

<style scoped>
.header-title {
  font-size: 32px;
  text-align: center;
  position: relative;
  margin-bottom: 20px;
}

.header-title::after {
  content: '';
  display: block;
  width: 60px;
  height: 4px;
  background-color: #FFA900;
  margin: 0 auto;
  margin-top: 10px;
}

.oval-button {
  border-radius: 25px;
  background-color: white;
  color: black;
  border: 1px solid black;
  transition: background-color 0.3s, color 0.3s;
}

.oval-button:hover {
  background-color: #FFA900;
  color: white;
}

.hover-card {
  transition: transform 0.3s;
  border-radius: 8px;
  overflow: hidden;
}

.hover-card:hover {
  transform: scale(1.05);
}

.cart-icon {
  margin-left: auto;
  background-color: transparent;
}

.sold-info {
  font-size: 14px;
  /* Adjust font size as needed */
  color: gray;
  /* Optional: change color to differentiate */
  margin-top: -27px;
  /* Space between name and sold quantity */
}

.see-more {
  margin-left: 7px;
  font-size: 14px;
  cursor: pointer;
}

.buy-now-btn {
  color: white;
  background-color: #FFA900;
  font-weight: bold;
  border-radius: 8px;
}

.see-more-btn {
  color: white;
  border-radius: 25px;
}
</style>
