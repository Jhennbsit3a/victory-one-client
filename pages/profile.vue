<template>
  <v-container>
    <v-row>
      <v-snackbar
        v-model="snackbar"
        top
        elevation="24"
      >
        {{ inform }}

        <template v-slot:action="{ attrs }">
          <v-btn
            color="pink"
            text
            v-bind="attrs"
            @click="closeInform"
          >
            Close
          </v-btn>
        </template>
      </v-snackbar>
      <!-- Sidebar -->
      <v-col cols="12" md="3">
        <v-card class="pa-5 profile-sidebar elevation-2">
          <!-- Profile Picture and Name -->
          <div class="profile-header text-center">
            <v-avatar size="80px" class="mb-4">
              <v-img v-if="user.profilePicture" :src="user.profilePicture" />
              <v-icon v-else style="font-size: 80px; color: #ccc;">mdi-account-circle</v-icon>
            </v-avatar>
            <h3>{{ user.name }}</h3>

            <!-- Button Container for Upload and Remove Profile -->
            <div class="button-container">
              <v-btn color="primary" @click="$refs.fileInput.click()" text style="font-size: 12px;">Upload
                Profile</v-btn>
              <input type="file" ref="fileInput" @change="uploadAvatar" accept="image/*" style="display: none;" />
              <v-btn color="error" @click="removeAvatar" text style="font-size: 12px;">Remove Profile</v-btn>
            </div>
          </div>

          <v-divider class="my-4"></v-divider>

          <!-- Sidebar Links -->
          <v-list dense>
            <v-list-item @click="navigateTo('/profile')" class="sidebar-item">
              <v-icon left style="font-size: 28px;">mdi-account-outline</v-icon>
              <v-list-item-title>My Account</v-list-item-title>
            </v-list-item>
            <v-list-item @click="navigateTo('/cart')" class="sidebar-item">
              <v-icon left style="font-size: 28px;">mdi-package-variant-closed</v-icon>
              <v-list-item-title>My Cart</v-list-item-title>
            </v-list-item>
            <v-list-item @click="navigateTo('/status')" class="sidebar-item">
              <v-icon left style="font-size: 28px;">mdi-truck-fast-outline</v-icon>
              <v-list-item-title
                style="display: flex; justify-content: space-between; align-items: center; width: 100%;">
                <span>My Orders</span>
                <v-badge v-if="hasOrders" color="red" content="!" overlap
                  style="margin-bottom: 15px; margin-right: 15px; margin-top: 10px;"></v-badge>
              </v-list-item-title>
            </v-list-item>
            <v-list-item @click="navigateTo('/transaction')" class="sidebar-item">
              <v-icon left style="font-size: 28px;">mdi-currency-usd</v-icon>
              <v-list-item-title>My Transactions</v-list-item-title>
            </v-list-item>
            <v-list-item @click="navigateTo('/rating')" class="sidebar-item">
              <v-icon left style="font-size: 28px;">mdi-star</v-icon>
              <v-list-item-title>Ratings</v-list-item-title>
            </v-list-item>
            <v-list-item @click="navigateTo('/privacy')" class="sidebar-item">
              <v-icon left style="font-size: 28px;">mdi-form-select</v-icon>
              <v-list-item-title>Privacy and Policy</v-list-item-title>
            </v-list-item>
            <v-list-item @click="navigateTo('/support')" class="sidebar-item">
              <v-icon left style="font-size: 28px;">mdi-help-circle-outline</v-icon>
              <v-list-item-title>Support</v-list-item-title>
            </v-list-item>
            <v-list-item @click="navigateTo('/change-password')" class="sidebar-item">
              <v-icon left style="font-size: 28px;">mdi-lock-outline</v-icon>
              <v-list-item-title>Change Password</v-list-item-title>
            </v-list-item>
            <v-list-item @click="showLogoutDialog = true" class="sidebar-item">
              <v-icon left style="font-size: 28px;">mdi-logout</v-icon>
              <v-list-item-title>Log Out</v-list-item-title>
            </v-list-item>
          </v-list>
        </v-card>
      </v-col>

      <!-- Main Content: Account Overview -->
      <v-col cols="12" md="9">
        <v-card class="pa-6 elevation-2">
          <!-- Title -->
          <v-card-title>
            <h2 class="headline mb-4">Account Overview</h2>
          </v-card-title>

          <v-divider class="my-4"></v-divider>

          <!-- Account Information Overview -->
          <v-list dense>
            <v-list-item>
              <v-list-item-content>
                <v-list-item-title>
                  <v-icon left color="#ffa900" style="font-size: 36px;">mdi-account-circle-outline</v-icon>
                  <span class="header-title">Account Details</span>
                </v-list-item-title>
                <v-list-item-subtitle class="info-title">
                  <span class="infos-title">Name: {{ user.name }} <br /> </span>
                  <span class="infos-title">Email: {{ user.email }} <br /> </span>
                  <span class="infos-title">Phone: {{ user.phone }} </span>
                </v-list-item-subtitle>
              </v-list-item-content>

              <v-list-item-action>
                <v-btn style="background-color: #ffa900; color: white;" @click="openEditInfoDialog">
                  Edit Info
                </v-btn>

              </v-list-item-action>
            </v-list-item>

            <v-divider class="my-4"></v-divider>

            <v-list-item>
              <v-list-item-content>
                <!-- Email Verification Header -->
                <v-list-item-title>
                  <v-icon left color="#ffa900" style="font-size: 36px;">mdi-email-check-outline</v-icon>
                  <span class="header-title">Email Verification</span>
                </v-list-item-title>
                <!-- Email Verification Subtitle -->
                <v-list-item-subtitle>
                  <span class="info-title">
                    {{ emailVerified ? 'Your email is verified.' : 'Your email is not verified.' }}
                  </span>
                </v-list-item-subtitle>
              </v-list-item-content>

              <!-- Verify Email Button -->
              <v-list-item-action>
                <v-btn 
                  style="background-color: #ffa900; color: white;" 
                  :disabled="emailVerified === true"
                  @click="verifyEmail"
                >
                  {{ emailVerified ? "Verified" : "Verify Email" }}
                </v-btn>
              </v-list-item-action>
            </v-list-item>

            <v-dialog v-model="showVerifyDialog" persistent max-width="400px">
              <v-card>
                <v-card-title class="headline">Email Verification</v-card-title>
                <v-card-text>
                  <p v-if="emailVerified">Your email is already verified!</p>
                  <p v-else>
                    Please check your inbox for a verification email. If you haven't received it, click the button below
                    to resend it.
                  </p>
                </v-card-text>
                <v-card-actions>
                  <v-btn color="green darken-1" text @click="showVerifyDialog = false">Close</v-btn>
                  <v-btn color="blue darken-1" text @click="sendVerificationEmail">Send Verification</v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>

            <!-- Edit Info Dialog -->
            <v-dialog v-model="showEditInfoDialog" max-width="500px">
              <v-card>
                <v-card-title>
                  <span class="headline">Edit Account Information</span>
                </v-card-title>

                <v-card-text>
                  <v-form ref="editInfoForm">
                    <!-- Separate Fields for First Name and Last Name -->
                    <v-text-field label="First Name" v-model="user.firstName" prepend-icon="mdi-account"
                      required></v-text-field>
                    <v-text-field label="Last Name" v-model="user.lastName" prepend-icon="mdi-account"
                      required></v-text-field>
                    <v-text-field label="Email" v-model="user.email" prepend-icon="mdi-email" required
                      type="email"></v-text-field>
                    <v-text-field label="Phone" v-model="user.phone" prepend-icon="mdi-phone" required></v-text-field>
                  </v-form>
                </v-card-text>

                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="primary" @click="saveEditInfo">Save</v-btn>
                  <v-btn @click="showEditInfoDialog = false">Cancel</v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>

            <v-divider class="my-4"></v-divider>

            <!-- <v-list-item>
              <v-list-item-content>
                <v-list-item-title>
                  <v-icon left color="#ffa900" style="font-size: 36px;">mdi-home-outline</v-icon>
                  <span class="header-title">Your Address</span>
                </v-list-item-title>
                <v-list-item-subtitle>
                  <span class="info-title">
                    {{ address }}
                  </span>
                </v-list-item-subtitle>
              </v-list-item-content>
              <v-list-item-action>
                <v-btn @click="showAddressDialog = true" style="background-color: #ffa900; color: white;">
                  Add Address
                </v-btn>
              </v-list-item-action>
            </v-list-item> -->

            <v-list-item>
              <v-list-item-content>
                <v-list-item-title>
                  <v-icon left color="#ffa900" style="font-size: 36px;">mdi-home-outline</v-icon>
                  <span class="header-title">Your Address</span>
                </v-list-item-title>
                <v-list-item-subtitle>
                  <span class="info-title">
                    {{ address || "No address found" }}
                  </span>
                </v-list-item-subtitle>
              </v-list-item-content>
              <v-list-item-action>
                <v-btn @click="openUpdateAddressDialog" style="background-color: #ffa900; color: white;">
                  Update Address
                </v-btn>
              </v-list-item-action>
            </v-list-item>

            <v-divider class="my-4"></v-divider>

            <!-- New Section for Saved Addresses -->
            <!-- <v-list-item>
              <v-list-item-content>
                <v-list-item-title>
                  <v-icon left color="#ffa900" style="font-size: 36px;">mdi-map-marker-outline</v-icon>
                  <span class="header-title">Manage Saved Addresses</span>
                </v-list-item-title>
              </v-list-item-content>
              <v-list-item-action>
                <v-btn @click="showSavedAddressesDialog = true" style="background-color: #ffa900; color: white;">
                  View Saved Addresses
                </v-btn>
              </v-list-item-action>
            </v-list-item> -->

            <!-- <v-divider class="my-4"></v-divider> -->

            <!-- Confirmation Dialog -->
            <v-dialog v-model="showLogoutDialog" max-width="400">
              <v-card>
                <v-card-title class="headline">Confirm Logout</v-card-title>
                <v-card-text>
                  Are you sure you want to log out?
                </v-card-text>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="primary" text @click="confirmLogout">Yes</v-btn>
                  <v-btn color="secondary" text @click="showLogoutDialog = false">Cancel</v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>

            <!-- Manage Address Dialog -->
            <v-dialog v-model="showAddressDialog" max-width="500">
              <v-card>
                <v-card-title>
                  <span class="headline">Update Address</span>
                </v-card-title>

                <v-card-text>
                  <v-form ref="addressForm">
                    <!-- Horizontal Layout for Province and City -->
                    <v-row>
                      <v-col cols="12" md="6">
                        <v-text-field label="Province" v-model="addressForm.province" prepend-icon="mdi-map-marker"
                          required></v-text-field>
                      </v-col>

                      <v-col cols="12" md="6">
                        <v-text-field label="City" v-model="addressForm.city" prepend-icon="mdi-city"
                          required></v-text-field>
                      </v-col>
                    </v-row>

                    <!-- Vertical Layout for Street and Zip Code -->
                    <v-text-field label="Street" v-model="addressForm.street" prepend-icon="mdi-map"
                      required></v-text-field>

                    <v-text-field label="Zip Code" v-model="addressForm.zip" prepend-icon="mdi-mailbox"
                      required></v-text-field>
                  </v-form>
                </v-card-text>

                <v-card-actions>
                  <v-spacer></v-spacer>
                  <!-- <v-btn color="primary" @click="updateAddress">Update</v-btn> -->
                   <v-btn color="primary" @click="updateAddress" :disabled="loadingUpdateAddress">
                    <v-progress-circular v-if="loadingUpdateAddress" indeterminate size="20" color="white"></v-progress-circular>
                    <span v-else>Update</span>
                  </v-btn>
                  <v-btn style="background-color: firebrick; color: white;" @click="resetAddressForm">Reset</v-btn>
                  <v-btn @click="showAddressDialog = false">Cancel</v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>


          </v-list>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import { auth, firestore } from '~/plugins/firebase'; // Ensure firebase is correctly imported
import { onAuthStateChanged, sendEmailVerification } from 'firebase/auth';
import { doc, addDoc, getDoc, deleteDoc, collection, getDocs, query, where, updateDoc, onSnapshot } from 'firebase/firestore';
import { getStorage, ref, uploadBytes, getDownloadURL } from 'firebase/storage';

export default {
  data() {
    return {
      user: {
        profilePicture: '',
        firstName: '',
        lastName: '',
        email: '',
        phone: '',
        address: '',
        savedAddresses: [],
      },
      inform: '',
      snackbar: false,
      loadingUpdateAddress: false,
      address: '',
      hasOrders: false, // New property to track if user has orders
      showEditInfoDialog: false,  // New property to control Edit Info modal visibility
      showAddressDialog: false,
      showSavedAddressesDialog: false,
      addressForm: {
        province: '',
        city: '',
        street: '',
        zip: '',
      },
      // verified: 'Verified'
      emailVerified: false,  // Track if the email is verified
      showVerifyDialog: false,  // Control visibility of the verify email dialog
      idImage: null,            // To store the uploaded ID image
      userImage: null,          // To store the uploaded user photo with ID // Store the ID entered by the user
      addressTableHeaders: [
        { text: 'Address', value: 'address' },
        { text: 'Actions', value: 'actions', sortable: false, align: 'center' },
      ],
      showLogoutDialog: false,
      loading: false,           // Add the loading property here to make it reactive
    };
  },
  async mounted() {
    
    onAuthStateChanged(auth, async (user) => {
      if (user) {
        // The user is authenticated
        // this.user.email = user.email;
        this.emailVerified = user.emailVerified;  // Update the emailVerified status

        // Fetch the user data from Firestore
        const userRef = doc(firestore, 'Users', user.uid);
        const userDoc = await getDoc(userRef);
              // Displaying firestore Data for checking
        const ordersRef = collection(firestore, "Orders");

        const querySnapshot = await getDocs(ordersRef);
          
          const ordersData = [];
          querySnapshot.forEach((doc) => {
            ordersData.push({ id: doc.id, ...doc.data() });
          });
          // console.table(user.uid)
            // Log deliveryAddress for each order
          ordersData.some((order) => {
            if(order.userId === user.uid){
              // console.log(`Order ID: ${order.userId}, Delivery Address: ${order.deliveryAddress}`);
              this.address = order.deliveryAddress;
              // console.log(this.user.address)
              return true;
            }
          });
        // Displaying firestore Data for checking

        if (userDoc.exists()) {
          const userData = userDoc.data();
          this.user.name = user.displayName || `${userData.firstName} ${userData.lastName}`;
          this.user.phone = userData.phone || '';
          this.user.email = userData.email;
          this.user.profilePicture = userData.profilePicture || user.photoURL;
          // console.log(userDoc.data())
        } else {
          console.log('No user document found in Firestore!');
        }

        // Real-time listener for user document (optional if you want to track real-time changes)
        onSnapshot(userRef, (snapshot) => {
          if (snapshot.exists()) {
            const userData = snapshot.data();
            this.user.name = user.displayName || `${userData.firstName} ${userData.lastName}`;
            this.user.phone = userData.phone || '';
            this.user.address = userData.address || '';
            this.user.profilePicture = userData.profilePicture || user.photoURL;

            // Fetch saved addresses for the current user
            this.loadSavedAddresses(user.uid);
          }
        });
      } else {
        console.log('User not authenticated');
      }
    });
    
  },
  methods: {
  openUpdateAddressDialog() {
    if (this.user.savedAddresses.length > 0) {
      const existingAddress = this.user.savedAddresses[0]; // Assuming the user has only one saved address
      this.addressForm = {
        id: existingAddress.id,
        province: existingAddress.province,
        city: existingAddress.city,
        street: existingAddress.street,
        zip: existingAddress.zip,
      };
    } else {
      // If no address exists, initialize an empty form
      this.addressForm = {
        id: '',
        province: '',
        city: '',
        street: '',
        zip: '',
      };
    }

    this.showAddressDialog = true;
  },
    async updateAddress() {
      try {
        this.loadingUpdateAddress = true; // Show loading indicator

        const user = auth.currentUser;
        if (!user) {
          console.error("User not authenticated.");
          this.loadingUpdateAddress = false;
          return;
        }

        const newAddress = `${this.addressForm.street}, ${this.addressForm.city}, ${this.addressForm.province}, ${this.addressForm.zip}`;

        // Update Orders collection (deliveryAddress)
        const ordersRef = collection(firestore, "Orders");
        const q = query(ordersRef, where("userId", "==", user.uid));
        const querySnapshot = await getDocs(q);

        if (querySnapshot.empty) {
          console.error("No orders found for this user.");
          this.loadingUpdateAddress = false;
          return;
        }

        const updatePromises = querySnapshot.docs.map((orderDoc) => {
          const orderRef = doc(firestore, "Orders", orderDoc.id);
          return updateDoc(orderRef, { deliveryAddress: newAddress });
        });

        await Promise.all(updatePromises);
        console.log("Delivery address updated successfully!");

        // Update Users collection (if applicable)
        const userRef = doc(firestore, "Users", user.uid);
        await updateDoc(userRef, { address: newAddress });

        console.log("User profile address updated successfully!");

        // Refresh UI in real-time
        this.address = newAddress;

        // Listen for real-time updates
        onSnapshot(userRef, (snapshot) => {
          if (snapshot.exists()) {
            this.address = snapshot.data().address;
          }
        });

        // Close dialog after successful update
        this.showAddressDialog = false;
      } catch (error) {
        console.error("Error updating address:", error);
      } finally {
        this.loadingUpdateAddress = false; // Hide loading indicator
      }
    },
    confirmLogout() {
      this.showLogoutDialog = false; // Close the dialog
      this.signOut(); // Proceed with the logout
    },
    // Method to trigger email verification
    async verifyEmail() {
      const currentUser = auth.currentUser;
      if (currentUser) {
        // If the email is already verified, show the dialog with the confirmation
        if (currentUser.emailVerified) {
          this.emailVerified = true;
          this.showVerifyDialog = true;
        } else {
          this.showVerifyDialog = true;
        }
      } else {
        alert('No user is logged in.');
      }
    },

    // Method to send a verification email
    async sendVerificationEmail() {
      const currentUser = auth.currentUser;
      if (currentUser && !currentUser.emailVerified) {
        try {
          await sendEmailVerification(currentUser);
          this.snackbar = true
          this.inform = "Verification email sent. Please check your inbox."
          // alert('Verification email sent. Please check your inbox.');
          this.emailVerified = false;  // Ensure it's not marked as verified yet
        } catch (error) {
          console.error('Error sending verification email:', error);
          this.snackbar = true
          this.inform = "Failed to send verification email."
          // alert('Failed to send verification email.');
        }
      }
    },
    async openEditInfoDialog() {
      try {
        const currentUser = auth.currentUser;

        if (currentUser) {
          // Fetch the user's document from Firestore
          const userRef = doc(firestore, 'Users', currentUser.uid);
          const userDoc = await getDoc(userRef);

          if (userDoc.exists()) {
            const userData = userDoc.data();
            // Populate the user object with fetched data
            this.user.firstName = userData.firstName || '';
            this.user.lastName = userData.lastName || '';
            this.user.email = userData.email || '';
            this.user.phone = userData.phone || '';
          } else {
            console.error('No user document found!');
          }
        } else {
          console.error('No user is logged in.');
        }

        // Show the dialog
        this.showEditInfoDialog = true;
      } catch (error) {
        console.error('Error fetching user data:', error);
      }
    },
    async checkUserOrders() {
      try {
        const user = auth.currentUser;
        if (user) {
          const q = query(
            collection(firestore, 'Orders'),
            where('userId', '==', user.uid)
          );
          const querySnapshot = await getDocs(q);

          // Update the hasOrders property based on the result
          this.hasOrders = !querySnapshot.empty;
        }
      } catch (error) {
        console.error('Error checking orders:', error);
      }
    },

    // // Your existing methods...a
    // async saveEditInfo() {
    //   // Your existing save logic...
    // },
    async saveEditInfo() {
      try {
        const user = auth.currentUser;
        if (!user) {
          // alert("You need to be logged in to edit your information.");
        this.snackbar = true
        this.inform = "You need to be logged in to edit your information."
          return;
        }

        // Update the user's information in Firestore
        const userRef = doc(firestore, 'Users', user.uid);
        await updateDoc(userRef, {
          firstName: this.user.firstName,
          lastName: this.user.lastName,
          email: this.user.email,
          phone: this.user.phone,
        });

        // Update the user data in the local state
        this.user.firstName = this.user.firstName;
        this.user.lastName = this.user.lastName;
        this.user.email = this.user.email;
        this.user.phone = this.user.phone;

        // Close the dialog
        this.showEditInfoDialog = false;
        this.snackbar = true
        this.inform = "Your information has been updated successfully!"

        // alert("Your information has been updated successfully!");
      } catch (error) {
        console.error("Error updating user info:", error);
        this.snackbar = true
        this.inform = "Failed to update your information."
        // alert("Failed to update your information.");
      }
    },
    async uploadAvatar(event) {
      const file = event.target.files[0];
      if (!file) return;

      const storage = getStorage();
      const user = auth.currentUser;

      if (!user) {
        this.snackbar = true
        this.inform = "You need to log in to upload an avatar."
        // alert("You need to log in to upload an avatar.");
        return;
      }

      try {
        const storageRef = ref(storage, `avatars/${user.uid}`);
        await uploadBytes(storageRef, file);
        const downloadURL = await getDownloadURL(storageRef);

        // Update the user's profile picture URL in Firestore
        const userRef = doc(firestore, 'Users', user.uid);
        await updateDoc(userRef, {
          profilePicture: downloadURL,
        });

        // Update the UI
        this.user.profilePicture = downloadURL;
        this.snackbar = true
        this.inform = "Avatar uploaded successfully!"

        // alert("Avatar uploaded successfully!");
      } catch (error) {
        console.error("Error uploading avatar:", error);
        this.snackbar = true
        this.inform = "Failed to upload avatar."
        // alert("Failed to upload avatar.");
      }
    },
    closeInform(){
      this.snackbar = false
    },
    async removeAvatar() {
      try {
        const user = auth.currentUser;
        if (!user) {
          // alert("You need to log in to remove your avatar.");
          this.snackbar = true
          this.inform = "You need to log in to remove your avatar."
          return;
        }

        // Update Firestore to reset profilePicture to default
        const userRef = doc(firestore, 'Users', user.uid);
        await updateDoc(userRef, {
          profilePicture: '', // Set profilePicture to an empty string
        });

        // Update the UI
        this.user.profilePicture = '';
        this.snackbar = true
        this.inform = "Avatar removed successfully!"
        // alert("Avatar removed successfully!");
      } catch (error) {
        console.error("Error removing avatar:", error);
        // alert("Failed to remove avatar.");
        this.snackbar = true
        this.inform = "Failed to remove avatar."
      }
    },
    async loadSavedAddresses(userId) {
      try {
        const q = query(collection(firestore, 'Address'), where('userID', '==', userId));
        const querySnapshot = await getDocs(q);

        this.user.savedAddresses = [];
        querySnapshot.forEach((doc) => {
          const data = doc.data();
          const address = `${data.street}, ${data.city}, ${data.province}, ${data.zip}`;
          this.user.savedAddresses.push({
            id: doc.id,  // Store the Firestore document ID
            address: address,  // Full address string
            province: data.province,
            city: data.city,
            street: data.street,
            zip: data.zip,
          });
        });
      } catch (error) {
        console.error('Error fetching saved addresses:', error);
      }
    },
    resetAddressForm() {
      this.addressForm = {
        province: '',
        city: '',
        street: '',
        zip: '',
      };
    },
    navigateTo(route) {
      this.$router.push(route);
    },
    signOut() {
      auth.signOut().then(() => {
        this.$router.push('/');
      }).catch((error) => {
        console.error('Error logging out:', error);
          this.snackbar = true
          this.inform = "Failed to log out. Please try again."
        // alert('Failed to log out. Please try again.');
      });
    },
    async editAddress(index) {
      const address = this.user.savedAddresses[index];
      this.addressForm = {
        province: address.province,
        city: address.city,
        street: address.street,
        zip: address.zip,
      };
      this.showAddressDialog = true;
    },
    async deleteAddress(index) {
      try {
        const user = auth.currentUser;
        if (user) {
          // Get the address document ID from the savedAddresses array
          const addressId = this.user.savedAddresses[index].id;

          // Delete the address from Firestore
          const addressRef = doc(firestore, 'Address', addressId);
          await deleteDoc(addressRef);

          // Remove the address from the local array (update UI)
          this.user.savedAddresses.splice(index, 1);
        }
      } catch (error) {
        console.error("Error deleting address:", error);
      }
    },

    async addAddress() {
      if (this.addressForm.province && this.addressForm.city && this.addressForm.street && this.addressForm.zip) {
        const newAddress = `${this.addressForm.street}, ${this.addressForm.city}, ${this.addressForm.province}, ${this.addressForm.zip}`;

        try {
          const user = auth.currentUser;
          if (user) {
            // Add address to Firestore
            const docRef = await addDoc(collection(firestore, 'Address'), {
              userID: user.uid,
              province: this.addressForm.province,
              city: this.addressForm.city,
              street: this.addressForm.street,
              zip: this.addressForm.zip,
            });

            // Push new address into user's savedAddresses array in the UI
            this.user.savedAddresses.push({
              id: docRef.id, // The document ID from Firestore
              address: newAddress, // Full address string
              province: this.addressForm.province,
              city: this.addressForm.city,
              street: this.addressForm.street,
              zip: this.addressForm.zip,
            });

            // Reset form fields
            this.addressForm = {
              province: '',
              city: '',
              street: '',
              zip: '',
            };
            this.showAddressDialog = false;
          } else {
            // alert('User is not authenticated');
          this.snackbar = true
          this.inform = "User is not authenticated"
          }
        } catch (error) {
          console.error('Error adding address:', error);
          // alert('Failed to add address.');
          this.snackbar = true
          this.inform = "Failed to add address."
          
        }
      } else {
        alert("Please fill out all fields.");
      }
    },
  },
};
</script>

<style scoped>
.header-title {
  font-size: 16px;
  font-weight: bold;
}

.profile-header {
  margin-bottom: 24px;
}

.button-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 10px;
}

.button-container .v-btn {
  margin: 5px 0;
}

.sidebar-item {
  cursor: pointer;
  margin-bottom: 16px;
}

.headline {
  font-size: 32px;
  font-weight: bold;
}

.v-icon {
  font-size: 20px;
}

.v-btn {
  font-weight: bold;
  font-size: 16px;
}

.header-title {
  font-size: 16px;
  font-weight: bold;
}

.info-title {
  font-size: 12px;
}

.infos-title {
  font-size: 12px;
  color: #333;
}

.profile-header {
  margin-bottom: 24px;
}

.sidebar-item {
  cursor: pointer;
  margin-bottom: 16px;
}

.headline {
  font-size: 32px;
  font-weight: bold;
}

.v-icon {
  font-size: 20px;
}

.v-btn {
  font-weight: bold;
  font-size: 16px;
}
</style>