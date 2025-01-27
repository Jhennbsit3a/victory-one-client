<template>
  <v-app>
    <v-container>
      <!-- MAIN PRODUCTS Header -->
      <v-row class="mb-4">
        <v-col cols="12">
          <h2 class="header-title">OUR PRODUCTS</h2>
        </v-col>
      </v-row>

      <!-- Search Bar for Products (Improved Design) -->
      <v-row>
        <v-col cols="12" md="6"> <!-- Adjust width for better balance -->
          <v-text-field v-model="searchTerm" placeholder="Search for products..." clearable @input="filterProducts"
            class="search-bar" prepend-inner-icon="mdi-magnify" outlined>
          </v-text-field>
        </v-col>
      </v-row>

      <!-- Layout for Categories and Product Grid -->
      <v-row>
        <!-- Category List on the Left with Header and Background Image -->
        <v-col cols="12" md="3">
          <v-card class="category-section">
            <div class="category-header-container">
              <h2 class="category-header">Product Categories</h2>
            </div>
            <v-list>
              <v-list-item-group>
                <v-list-item v-for="category in categories" :key="category.id" @click="selectCategory(category)"
                  :class="{ 'selected-category': selectedCategory && selectedCategory.CategoryID === category.CategoryID }">
                  <v-list-item-icon>
                    <v-icon
                      :class="{ 'selected-icon': selectedCategory && selectedCategory.CategoryID === category.CategoryID }">mdi-plus</v-icon>
                  </v-list-item-icon>
                  <v-list-item-title>{{ category.ProductType }}</v-list-item-title>
                </v-list-item>
              </v-list-item-group>
            </v-list>
          </v-card>
        </v-col>

        <!-- Product Grid on the Right -->
        <v-col cols="12" md="9">
          <v-row>
            <v-col v-for="product in filteredProducts" :key="product.id" cols="12" md="4">
              <v-card class="hover-card" style="position: relative;" @click="goToProduct(product.id)">
                <!-- Add click event on the card -->
                <v-img :src="product.image" height="200px"></v-img>
                <v-card-title class="d-flex justify-space-between">
                  <span>{{ product.name }}</span>
                  <!-- Cart icon next to product name -->
                  <v-btn icon class="cart-icon" @click.stop="addToCart(product, quantity)" :disabled="loading">
                    <v-icon style="color: #FFA900;">mdi-cart-minus</v-icon>
                  </v-btn>
                </v-card-title>

                <v-card-subtitle style="color: black;">₱{{ product.price }}</v-card-subtitle>

                <!-- Sold Quantity Display -->
                <v-card-subtitle class="sold-info" style="color: black;">Sold: {{ product.soldQuantity
                  }}</v-card-subtitle>

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
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script>
import { firestore,auth } from '~/plugins/firebase';
import { collection, getDocs, onSnapshot, addDoc, where, updateDoc,query } from 'firebase/firestore'; // Import doc, updateDoc, and increment
import { getAuth } from 'firebase/auth'; // Import Firebase auth

export default {
  data() {
    return {
      loading: false,  // Add a loading state
      products: [],
      filteredProducts: [],
      categories: [],
      searchTerm: '',
      selectedCategory: null,
      quantity: 1,
    };
  },
  async created() {
    try {
      // Fetch categories from Firestore
      const categoriesSnapshot = await getDocs(collection(firestore, 'Categories'));
      this.categories = categoriesSnapshot.docs.map(doc => ({
        id: doc.id,
        ProductType: doc.data().ProductType,
        CategoryID: doc.data().CategoryID,
      }));

      // Fetch products from Firestore
      const productsRef = collection(firestore, 'Products');
  onSnapshot(productsRef, (snapshot) => {
    this.products = snapshot.docs.map(doc => ({
      id: doc.id,
      name: doc.data().ProductName,
      price: doc.data().Price,
      image: doc.data().Image,
      categoryID: doc.data().CategoryID,
      soldQuantity: doc.data().Sold || 0,
    }));
    this.filterProducts();
  });

      // Fetch orders from Firestore
      const ordersSnapshot = await getDocs(collection(firestore, 'Orders'));
      const orders = ordersSnapshot.docs.map(doc => doc.data());

      // Calculate total sold quantity for each product
      this.products = this.products.map(product => {
        const productId = product.id;

        // Calculate total quantity sold for this product
        const totalSold = orders.reduce((sum, order) => {
          const productInCart = order.cartItems.find(item => item.productID === productId);
          if (productInCart) {
            return sum + productInCart.Quantity;
          }
          return sum;
        }, 0);

        // Update soldQuantity field
        product.soldQuantity = totalSold;

        return product;
      });

      // Initialize filteredProducts
      this.filteredProducts = this.products;

      // Check for category in query parameters
      const selectedCategory = this.$route.query.category;
      if (selectedCategory) {
        this.selectedCategory = this.categories.find(cat => cat.ProductType === selectedCategory);
      }
      this.filterProducts(); // Call filterProducts after fetching data
    } catch (error) {
      console.error("Error fetching data: ", error);
    }
  },
  methods: {
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

          console.log('Product added to cart:', product.ProductName); // Log success message
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
    goToProduct(productId) {
      this.$router.push(`/product/${productId}`);
    },
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
        productName: product.ProductName,
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
    selectCategory(category) {
      if (this.selectedCategory && this.selectedCategory.CategoryID === category.CategoryID) {
        this.selectedCategory = null;
      } else {
        this.selectedCategory = category;
      }
      this.filterProducts();
    },
    filterProducts() {
      const search = this.searchTerm ? this.searchTerm.toLowerCase() : '';
      this.filteredProducts = this.products.filter(product => {
        const matchesName = product.name.toLowerCase().includes(search);
        const matchesCategory = this.selectedCategory ? product.categoryID === this.selectedCategory.CategoryID : true;
        return matchesName && matchesCategory;
      });
      // console.log(`Filtered Products: ${JSON.stringify(this.filteredProducts)}`);
    }
  },
};
</script>

<style scoped>
.header-title {
  font-size: 32px;
  font-family: 'Sequel Sans', sans-serif;
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

.category-header-container {
  background-image: url('@/assets/carousel 3.jpg');
  background-size: cover;
  background-position: center;
  padding: 30px 0;
  text-align: center;
  border-radius: 8px 8px 0 0;
}

.category-header {
  font-size: 24px;
  font-weight: bold;
  color: white;
  margin: 0;
}

.category-section {
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 0;
}

.category-section v-list {
  padding: 16px;
}

.selected-category {
  background-color: #FFA900;
  /* Highlight color */
  color: white;
  /* Text color for visibility */
}

.selected-icon {
  color: white;
  /* Change icon color to white when selected */
}

.hover-card {
  transition: transform 0.3s;
  border-radius: 8px;
  overflow: hidden;
  cursor: pointer;
  /* Change cursor to pointer for better UX */
}

.hover-card:hover {
  transform: scale(1.05);
}

.cart-icon-btn {
  margin: 0;
  padding: 0;
}

.search-bar {
  width: 100%;
}

.buy-now-btn {
  color: white;
  background-color: #FFA900;
  font-weight: bold;
  border-radius: 8px;
}

.see-more {
  margin-left: 7px;
  margin-top: -20px;
  font-size: 14px;
  cursor: pointer;
}

.sold-info {
  font-size: 14px;
  /* Adjust font size as needed */
  color: gray;
  /* Optional: change color to differentiate */
  margin-top: -27px;
  /* Space between name and sold quantity */
}

.plus-icon {
  font-size: 20px;
  color: #FFA900;
}
</style>
