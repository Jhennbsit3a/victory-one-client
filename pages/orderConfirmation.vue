<template>
  <div class="order-confirmation">
    <h2>Order Confirmation</h2>

    <!-- Loading Spinner -->
    <div v-if="loading" class="loading-overlay">
      <v-progress-circular indeterminate color="primary" size="50" width="5"
        class="loading-spinner"></v-progress-circular>
    </div>

    <div v-if="orderData" class="order-summary">
      <!-- Order Items Section -->
      <div class="order-items">
        <h3>Items</h3>
        <ul>
          <li v-for="item in orderData.cartItems" :key="item.id" class="order-item">
            <div class="item-image">
              <img :src="item.image" alt="Product Image" class="product-image" v-if="item.image" />
            </div>
            <div class="item-details">
              <p class="product-name"><strong>{{ item.productName }}</strong></p>
              <p>Price: ₱{{ item.price.toFixed(2) }}</p>
              <p>Quantity: {{ item.Quantity }}</p>
            </div>
          </li>
        </ul>
      </div>

      <!-- Order Details Section -->
      <div class="order-details">
        <h3>Order Details</h3>
        <p><strong>Delivery Address:</strong> {{ orderData.deliveryAddress }}</p>
        <p><strong>Estimated Delivery Date:</strong> {{ orderData.estimatedDeliveryDate }}</p>
        <p><strong>Payment Method:</strong> {{ orderData.paymentMethod }}</p>

        <!-- Separation Line -->
        <hr class="separator" />

        <!-- Subtotal, Tax, and Total Section -->
        <div class="pricing-details">
          <div>
            <p><strong>Subtotal:</strong> ₱{{ orderData.subtotal.toFixed(2) }}</p>

            <p v-if="paymentMethod !== 'Pick up'">
              <strong>Shipping Fee:</strong> ₱{{ orderData.tax.toFixed(2) }}
            </p>

            <p><strong>Total:</strong> ₱{{ orderData.total.toFixed(2) }}</p>
          </div>
        </div>
      </div>

      <!-- Action Buttons -->
      <div class="action-buttons">
        <button @click="openDialog" class="confirm-button">Confirm Order</button>
        <button @click="goBacktoCart" class="go-back-button">Go Back to Cart</button>
      </div>

      <!-- GCash Info Dialog -->
      <!-- <v-dialog v-model="gcashDialog" max-width="500px">
        <v-card>
          <v-card-title class="headline">GCash Payment</v-card-title>
          <v-card-text>
            <div class="text-center">
              <img src="@/assets/Gcash.jpg" alt="GCash Image" class="gcash-image" />
              <p class="gcash-description">Scan the QR code to pay via GCash.</p>
              <p class="gcash-description">Input Reference Number.<input type="text"/></p>
            </div>
          </v-card-text>
          <v-card-actions>
            <v-btn @click="confirmOrder" color="green">Done</v-btn>
            <v-btn @click="closeGcashDialog" color="red">Cancel</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog> -->
        <!-- GCash Info Dialog -->
      <v-dialog v-model="gcashDialog" max-width="500px">
        <v-card>
          <v-card-title class="headline">GCash Payment</v-card-title>
          <v-card-text>
            <div class="text-center">
              <p class="gcash-instruction">
                Please follow the instructions below before proceeding with the payment:
              </p>
              <ol class="gcash-steps">
                <li>Scan the QR code below using your GCash app.</li>
                <li>Enter the exact payment amount and complete the transaction.</li>
                <li>Take a screenshot of the transaction receipt for reference.</li>
                <li>
                  Locate the <strong>Reference Number</strong> on your GCash receipt.  
                  This is a 13-digit number usually found in the transaction details.
                </li>
                <li>Enter the <strong>Reference Number</strong> in the field below before proceeding.</li>
              </ol>
              <img src="@/assets/Gcash.jpg" alt="GCash Image" class="gcash-image" />

              <!-- Input Fields -->
              <v-text-field 
                v-model="referenceNumber" 
                label="Reference Number"
                outlined
                dense
                maxlength="13"
                type="text"
                placeholder="Enter 13-digit reference number"
                @input="validateReferenceNumber"
              ></v-text-field>
              <v-text-field 
                v-model="totalAmount" 
                label="Total amount to pay" 
                outlined 
                dense
                :disabled="true"
                required
              ></v-text-field>
              <!-- <v-text-field 
                v-model="receiptNumber" 
                label="GCash Receipt Number" 
                outlined 
                dense
                required
              ></v-text-field> -->
            </div>
          </v-card-text>

          <v-card-actions class="justify-end">
            <v-btn @click="confirmOrder" color="green" :disabled="!isReferenceValid">
              Confirm Payment
            </v-btn>
            <v-btn @click="closeGcashDialog" color="red">Cancel</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>


      <!-- QR Code Dialog -->
      <v-dialog v-model="qrCodeDialog" max-width="400px">
        <v-card>
          <v-card-title class="headline">Order QR Code</v-card-title>
          <v-card-text>
            <div class="text-center">
              <canvas ref="qrCanvas" width="200" height="200"></canvas>
              <p class="qr-description">
                You can use this QR code for pickup or as proof of your order.
              </p>
            </div>
          </v-card-text>
          <v-card-actions>
            <v-btn @click="confirmOrder" color="green" style="color: white;">Done</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

    </div>
  </div>
</template>

<script>
import QRCode from 'qrcode';
import { firestore,storage } from '~/plugins/firebase';
// import { ref, uploadBytes, getDownloadURL } from "firebase/storage";
import { collection, addDoc, getDocs, query, where, updateDoc, doc } from 'firebase/firestore';
import { getAuth } from 'firebase/auth';

export default {
  data() {
    return {
      gcashDialog: false,            // Controls the GCash payment dialog visibility
      referenceNumber: "",      // Stores the GCash reference number
      // receiptNumber: "",        // Stores the GCash receipt number
      isReferenceValid: false,  // Controls the "Confirm Payment" button state
      orderData: null,
      qrCodeDialog: false, // Dialog visibility for QR code
      gcashDialog: false,  // Dialog visibility for GCash payment info
      orderId: null,       // Store the orderId after order creation
      loading: false,      // Loading state
      paymentMethod:'',
      inform:'',
      totalAmount:0,
    };
  },
  created() {
    const orderData = this.$route.query.orderData;
    if (orderData) {
      this.orderData = JSON.parse(orderData);
      this.fetchProductImages();
    }
    this.paymentMethod = this.orderData.paymentMethod
  },
  methods: {
    async fetchProductImages() {
      try {
        const productsRef = collection(firestore, 'Products');
        for (let item of this.orderData.cartItems) {
          const q = query(productsRef, where('productName', '==', item.productName));
          const querySnapshot = await getDocs(q);
          querySnapshot.forEach((doc) => {
            if (doc.exists()) {
              const productData = doc.data();
              item.image = productData.Image;
            }
          });
        }
      } catch (error) {
        console.error('Error fetching product images:', error);
      }
    },
    // async confirmOrder() {
    //   try {
    //     // this.loading = true; 
    //     const auth = getAuth();
    //     const user = auth.currentUser;

    //     if (!user) {
    //       console.error('User not authenticated.');
    //       this.loading = false;
    //       return;
    //     }

    //     if (this.orderData) {
    //       const orderRef = collection(firestore, 'Orders');
    //       const orderDocRef = await addDoc(orderRef, {
    //         userId: this.orderData.userId,
    //         cartItems: this.orderData.cartItems, // Only selected items
    //         deliveryAddress: this.orderData.deliveryAddress,
    //         paymentMethod: this.orderData.paymentMethod,
    //         subtotal: this.orderData.subtotal,
    //         tax: this.orderData.tax,
    //         total: this.orderData.total,
    //         estimatedDeliveryDate: this.orderData.estimatedDeliveryDate,
    //         status: 'Pending',
    //         createdAt: new Date(),
    //       });

    //       this.orderId = orderDocRef.id;
    //       this.orderData.orderId = this.orderId;

    //       // setTimeout(() => {
    //       //   if (this.orderData.paymentMethod === 'Gcash') {
    //       //     this.openGcashDialog();
    //       //   } else if (this.orderData.paymentMethod === 'Pick up') {
    //       //     this.openQrCodeDialog();
    //       //   } else {
    //       //     this.goBack();
    //       //   }
    //       //   this.loading = false;
    //       // }, 2000);

    //       // ✅ FIX: Only update selected items
    //       const cartRef = collection(firestore, "Cart");
    //       const selectedProductIDs = this.orderData.cartItems.map(item => item.productID);
    //       const q = query(cartRef, where("userID", "==", user.uid), where("ProductID", "in", selectedProductIDs));
    //       const querySnapshot = await getDocs(q);

    //       const updatePromises = querySnapshot.docs.map((docSnapshot) => {
    //         const docRef = doc(firestore, "Cart", docSnapshot.id);
    //         return updateDoc(docRef, { orderStatus: "Confirmed" });
    //       });

    //       await Promise.all(updatePromises);
    //       console.log("Order status updated successfully for selected items.");
    //       this.goBack();
    //     }
    //   } catch (error) {
    //     console.error('Error confirming order:', error);
    //     this.loading = false;
    //   }
    // },
        // Restrict input to numbers only and check length
    validateReferenceNumber() {
      this.referenceNumber = this.referenceNumber.replace(/\D/g, ""); // Remove non-numeric characters
      this.isReferenceValid = this.referenceNumber.length === 13; // Enable button only if 13 digits
    },
    async confirmOrder() {
      try {
        const auth = getAuth();
        const user = auth.currentUser;

        if (!user) {
          console.error('User not authenticated.');
          return;
        }

        if (!this.referenceNumber) {
          console.error('Reference Number are required.');
          return;
        }

        if (this.orderData) {
          const orderRef = collection(firestore, 'Orders');
          const orderDocRef = await addDoc(orderRef, {
            userId: this.orderData.userId,
            cartItems: this.orderData.cartItems, 
            deliveryAddress: this.orderData.deliveryAddress,
            paymentMethod: this.orderData.paymentMethod,
            subtotal: this.orderData.subtotal,
            tax: this.orderData.tax,
            total: this.orderData.total,
            estimatedDeliveryDate: this.orderData.estimatedDeliveryDate,
            status: 'Pending',
            referenceNumber: this.referenceNumber,  // ✅ Save Reference Number
            // gcashReceiptNumber: this.receiptNumber,      // ✅ Save Receipt Number
            createdAt: new Date(),
          });

          this.orderId = orderDocRef.id;
          this.orderData.orderId = this.orderId;

          // ✅ Update only selected cart items
          const cartRef = collection(firestore, "Cart");
          const selectedProductIDs = this.orderData.cartItems.map(item => item.productID);
          const q = query(cartRef, where("userID", "==", user.uid), where("ProductID", "in", selectedProductIDs));
          const querySnapshot = await getDocs(q);

          const updatePromises = querySnapshot.docs.map((docSnapshot) => {
            const docRef = doc(firestore, "Cart", docSnapshot.id);
            return updateDoc(docRef, { orderStatus: "Confirmed" });
          });

          await Promise.all(updatePromises);
          console.log("Order confirmed with GCash details.");

          this.closeGcashDialog();
          this.goBack();
        }
      } catch (error) {
        console.error('Error confirming order:', error);
      }
    },
    openDialog() {
        try {
          this.loading = true; 
          const auth = getAuth();
          const user = auth.currentUser;

          if (!user) {
            console.error('User not authenticated.');
            this.loading = false;
            return;
          }

          if (this.orderData) {
            // const orderRef = collection(firestore, 'Orders');
            // const orderDocRef = await addDoc(orderRef, {
            //   userId: this.orderData.userId,
            //   cartItems: this.orderData.cartItems, // Only selected items
            //   deliveryAddress: this.orderData.deliveryAddress,
            //   paymentMethod: this.orderData.paymentMethod,
            //   subtotal: this.orderData.subtotal,
            //   tax: this.orderData.tax,
            //   total: this.orderData.total,
            //   estimatedDeliveryDate: this.orderData.estimatedDeliveryDate,
            //   status: 'Pending',
            //   createdAt: new Date(),
            // });

            // this.orderId = orderDocRef.id;
            // this.orderData.orderId = this.orderId;
            this.totalAmount = this.orderData.total
            setTimeout(() => {
              if (this.orderData.paymentMethod === 'Gcash') {
                this.gcashDialog = true;
              } else if (this.orderData.paymentMethod === 'Pick up') {
                this.openQrCodeDialog();
              } else {
                this.goBack();
              }
              this.loading = false;
            }, 2000);

            // ✅ FIX: Only update selected items
            // const cartRef = collection(firestore, "Cart");
            // const selectedProductIDs = this.orderData.cartItems.map(item => item.productID);
            // const q = query(cartRef, where("userID", "==", user.uid), where("ProductID", "in", selectedProductIDs));
            // const querySnapshot = await getDocs(q);

            // const updatePromises = querySnapshot.docs.map((docSnapshot) => {
            //   const docRef = doc(firestore, "Cart", docSnapshot.id);
            //   return updateDoc(docRef, { orderStatus: "Confirmed" });
            // });

            // await Promise.all(updatePromises);
            // console.log("Order status updated successfully for selected items.");
          }
        } catch (error) {
          console.error('Error confirming order:', error);
          this.loading = false;
        }
      //   this.loading = true; 
      // // this.gcashDialog = true; // Show GCash dialog
      //     setTimeout(() => {
      //       this.gcashDialog = true; // Show GCash dialog
      //       this.loading = false;
      //     }, 2000);
    },
    closeGcashDialog() {
      this.gcashDialog = false; // Close GCash dialog
    },
    openQrCodeDialog() {
      this.gcashDialog = false; // Close GCash dialog
      this.qrCodeDialog = true; // Show QR code dialog
      this.$nextTick(() => {
        this.generateQrCode();
      });
    },
    closeQrCodeDialog() {
      this.qrCodeDialog = false;
    },
    async generateQrCode() {
      try {
        const usersRef = collection(firestore, 'Users');
        const q = query(usersRef, where('__name__', '==', this.orderData.userId));
        const querySnapshot = await getDocs(q);

        let userName = 'Unknown User';
        querySnapshot.forEach((doc) => {
          if (doc.exists()) {
            const userDetails = doc.data();
            userName = `${userDetails.firstName || ''} ${userDetails.lastName || ''}`.trim() || 'Unknown User';
          }
        });

        const orderDetails = this.orderData.orderId        
        const canvas = this.$refs.qrCanvas;

        if (canvas) {
          const ctx = canvas.getContext('2d');
          ctx.clearRect(0, 0, canvas.width, canvas.height);
          await QRCode.toCanvas(canvas, orderDetails, {
            width: 200,
            margin: 4,
          });
        }
      } catch (error) {
        console.error('Error generating QR code:', error);
      }
    },
    goBacktoCart() {
      this.$router.push({ name: 'cart' });
    },
    goBack() {
      this.$router.push({ name: 'orderConfirmationSuccess' });
    },
  },
};
</script>

<style scoped>
.gcash-instructions {
  font-size: 14px;
  color: #333;
  background: #f3f4f6;
  padding: 10px;
  border-radius: 8px;
  margin-bottom: 15px;
  text-align: left;
  line-height: 1.5;
}
.gcash-steps{
  text-align: left;
}
.loading-overlay {
  position: fixed;
  /* This makes the spinner float above other content */
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: rgba(255, 255, 255, 0.8);
  /* Slightly transparent background */
  z-index: 9999;
  /* Ensures the spinner stays on top */
}

.loading-spinner {
  margin: 0 auto;
}

.order-confirmation {
  padding: 20px;
  background-color: #fff;
  border-radius: 8px;
}

.gcash-image {
  width: 100%;
  height: auto;
  max-width: 300px;
  margin-bottom: 10px;
}

.gcash-description {
  font-size: 1.2rem;
  color: #444;
}

.separator {
  border: none;
  border-top: 2px solid #ddd;
  margin: 20px 0;
}

.order-summary {
  max-width: 1200px;
  margin: 0 auto;
}

h2 {
  font-size: 2rem;
  color: #333;
  margin-bottom: 20px;
}

h3 {
  margin-bottom: 15px;
  font-size: 1.5rem;
  color: #444;
}

.order-items,
.order-details {
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  padding: 20px;
  margin-bottom: 20px;
}

.order-item {
  display: flex;
  margin-bottom: 20px;
  border-bottom: 1px solid #ddd;
  padding-bottom: 15px;
}

.item-image {
  flex: 1;
  max-width: 120px;
}

.product-image {
  width: 100%;
  border-radius: 8px;
  object-fit: cover;
}

.item-details {
  flex: 2;
  padding-left: 20px;
}

.item-details p {
  margin: 5px 0;
}

.pricing-details {
  margin-top: 20px;
}

button {
  padding: 12px 20px;
  font-size: 1rem;
  border-radius: 5px;
  cursor: pointer;
  border: none;
}

.confirm-button {
  background-color: #28a745;
  color: white;
  margin-right: 15px;
}

.confirm-button:hover {
  background-color: #218838;
}

.go-back-button {
  background-color: #007bff;
  color: white;
}

.go-back-button:hover {
  background-color: #0056b3;
}

.qr-description {
  margin-top: 10px;
  font-size: 1rem;
  color: #444;
}
</style>
