<template>
  <v-container class="chat-container pa-4" style="max-width: 420px;">
    <!-- Header Title Section (Centered) -->
    <v-row justify="center">
      <v-col cols="12" class="title-btn text-center">
        <h3 class="chat-header">Ask Questions or Concerns</h3>
      </v-col>
    </v-row>

    <v-row>
      <v-col cols="12">
        <!-- Chat Box Section -->
        <div class="chat-box"
          style="height: 320px; overflow-y: auto; padding: 20px; background: #f9f9f9; border-radius: 15px;">
          <div v-for="(message, index) in messages" :key="index" class="mb-3">
            <div v-if="message.type === 'bot'" class="bot-message">
              <v-chip class="message-bubble bot-bubble" color="primary" dark>{{ message.text }}</v-chip>
            </div>
            <div v-else-if="message.type === 'typing'" class="typing-indicator">
              <v-chip class="message-bubble typing-bubble" color="grey lighten-2">Typing...</v-chip>
            </div>
            <div v-else class="user-message">
              <v-chip class="message-bubble user-bubble" color="teal" dark>{{ message.text }}</v-chip>
            </div>
          </div>
        </div>
      </v-col>

      <!-- Question Chips Section -->
      <v-col cols="12" class="mt-4">
        <div class="question-options">
          <v-chip v-for="(question, index) in currentQuestions" :key="index" class="mr-2 mb-2" color="blue lighten-2"
            dark @click="handleQuestionClick(question)">
            {{ question.text }}
          </v-chip>
        </div>
      </v-col>

      <!-- Input Box and Send Button -->
      <v-col cols="12" class="mt-4">
        <v-row align="center">
          <v-col cols="9">
            <v-text-field v-model="userInput" placeholder="Type your message..." outlined dense
              class="chat-input"></v-text-field>
          </v-col>
          <v-col cols="3">
            <v-btn color="primary" @click="handleSend" elevation="2" class="send-btn">
              <v-icon>mdi-send</v-icon>
            </v-btn>
          </v-col>
        </v-row>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import Fuse from "fuse.js";

export default {
  data() {
    return {
      messages: [],
      allQuestions: [
        {
          text: "What are your services?",
          response: "We provide ordering services and high-quality aluminum products, including seamless steel pipes, precision tubes, alloy pipes, and more.",
          next: ["What products do you offer?", "What are your operating hours?"],
        },
        {
          text: "What products do you offer?",
          response: "We offer seamless steel pipes, precision tubes, alloy pipes, copper tubes, aluminum tubes, and other products used in various industries.",
          next: ["What materials are your products made from?", "What are the applications of your products?"],
        },
        {
          text: "What materials are your products made from?",
          response: "Our products are made from high-quality raw materials like steel, copper, and aluminum, ensuring durability and reliability.",
          next: ["What products do you offer?", "What are the applications of your products?"],
        },
        {
          text: "What are the applications of your products?",
          response: "Our products are used in fluid transport, boilers, heat exchangers, construction, oil pipelines, and many other industries.",
          next: ["What products do you offer?", "What materials are your products made from?"],
        },
        {
          text: "Where are you located?",
          response: "We are located in San Matias, San Fernando, Pampanga.",
          next: ["Do you ship internationally?", "How long does shipping take?"],
        },
        {
          text: "Do you ship internationally?",
          response: "Currently, we do not offer international shipping. We only deliver within the Philippines.",
          next: ["Where are you located?", "How long does shipping take?"],
        },
        {
          text: "How long does shipping take?",
          response: "Shipping generally takes 3-7 business days depending on your location within the Philippines.",
          next: ["Do you offer free shipping?", "Can I track my order?"],
        },
        {
          text: "Can I track my order?",
          response: "You can track your order in your Profile under the 'My Order' section.",
          next: ["How long does shipping take?", "Do you offer free shipping?"],
        },
        {
          text: "Do you offer free shipping?",
          response: "Yes, we offer free shipping on orders over ₱1000.",
          next: ["How long does shipping take?", "Can I track my order?"],
        },
        {
          text: "What payment methods do you accept?",
          response: "We accept Gcash, Cash On Delivery (COD), and Pickup.",
          next: ["Do you offer any promotions or discounts?", "Can I cancel my order?"],
        },
        {
          text: "Can I cancel my order?",
          response: "Orders can be canceled within 24 hours of purchase.",
          next: ["What payment methods do you accept?", "How do I track my order?"],
        },
        {
          text: "What is your mission?",
          response: "Our mission is to provide the highest quality aluminum products and ensure customer satisfaction through reliable service and innovation.",
          next: ["What is your vision?", "What industries do you serve?"],
        },
        {
          text: "What is your vision?",
          response: "Our vision is to be a global leader in providing sustainable and high-performance aluminum products.",
          next: ["What is your mission?", "What industries do you serve?"],
        },
        {
          text: "What industries do you serve?",
          response: "We serve various industries, including construction, manufacturing, oil and gas, automotive, and more.",
          next: ["What is your mission?", "What products do you offer?"],
        },
        {
          text: "How long have you been in business?",
          response: "We have been in business for over 10 years, providing high-quality products and services to our customers.",
          next: ["What industries do you serve?", "Can I work with you?"],
        },
        {
          text: "Can I work with you?",
          response: "Yes, we are always looking for passionate individuals to join our team. Please visit our careers page for more information.",
          next: ["Are you hiring?", "How long have you been in business?"],
        },
        {
          text: "Are you hiring?",
          response: "We are currently hiring for several positions. Please check our website for open job listings.",
          next: ["Can I work with you?", "What is your mission?"],
        },
        {
          text: "How can I contact customer support?",
          response: "You can reach customer support by emailing support@victory-one.com or calling our hotline at 123-456-7890.",
          next: ["What is your refund policy?", "What is your return policy?"],
        },
        {
          text: "Do you have an FAQ page?",
          response: "Yes, we have an FAQ page where you can find answers to common questions. You can visit it on our website.",
          next: ["How can I contact customer support?", "What is your refund policy?"],
        },
        {
          text: "What is your refund policy?",
          response: "We offer refunds within 30 days of purchase, provided the product is in original condition.",
          next: ["How can I contact customer support?", "Do you have an FAQ page?"],
        },
        {
          text: "Do you offer any promotions or discounts?",
          response: "We frequently offer promotions and discounts. Please subscribe to our newsletter to stay updated.",
          next: ["What payment methods do you accept?", "Can I get a discount for bulk purchases?"],
        },
        {
          text: "Can I get a discount for bulk purchases?",
          response: "Yes, we offer discounts for bulk orders. Please contact us directly to discuss your requirements.",
          next: ["What payment methods do you accept?", "How can I place an order?"],
        },
      ],
      currentQuestions: [],
      userInput: "",
    };
  },
  created() {
    this.displayBotMessage("Hi! How can I help you today?", 2000);
    this.resetQuestions();

    // Initialize Fuse.js for fuzzy matching
    this.fuse = new Fuse(this.allQuestions, {
      keys: ["text"],  // Ensure we're searching the text property of questions
      threshold: 0.3, // Lower threshold for more lenient matching
      includeScore: true,
    });
  },
  methods: {
    resetQuestions() {
      this.currentQuestions = this.allQuestions.slice(0, 3);
    },
    handleQuestionClick(question) {
      this.messages.push({ type: "user", text: question.text });
      this.displayBotMessage(question.response);

      const nextQuestions = question.next.map((text) =>
        this.allQuestions.find((q) => q.text === text)
      );
      this.currentQuestions = nextQuestions.length ? nextQuestions : this.allQuestions.slice(0, 3);
    },
    handleSend() {
      if (this.userInput.trim()) {
        const userText = this.userInput.trim().toLowerCase(); // Ensure case-insensitive comparison
        this.messages.push({ type: "user", text: this.userInput });

        // Search for the best match using Fuse.js
        const result = this.fuse.search(userText);

        if (result.length > 0) {
          const matchedQuestion = result[0].item;
          this.displayBotMessage(matchedQuestion.response);

          const nextQuestions = matchedQuestion.next.map((text) =>
            this.allQuestions.find((q) => q.text === text)
          );
          this.currentQuestions = nextQuestions.length ? nextQuestions : this.allQuestions.slice(0, 3);
        } else {
          this.displayBotMessage("That's an interesting question! It's not available in the system right now, but feel free to ask anything else, and I'll do my best to assist you.");
        }

        this.userInput = "";
      }
    },
    displayBotMessage(messageText, delay = 3000) {
      this.messages.push({ type: "typing" });
      setTimeout(() => {
        this.messages.pop();
        this.messages.push({ type: "bot", text: messageText });
        this.$nextTick(() => {
          const chatBox = this.$el.querySelector(".chat-box");
          chatBox.scrollTop = chatBox.scrollHeight;
        });
      }, delay);
    },
  },
};
</script>


<style scoped>
.chat-container {
  background-color: #ffffff;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  border-radius: 20px;
  padding: 20px;
}

.chat-box {
  background-color: #f9f9f9;
  border-radius: 15px;
  padding: 20px;
  height: 320px;
  overflow-y: auto;
}

.message-bubble {
  padding: 12px 16px;
  font-size: 14px;
  max-width: 80%;
  border-radius: 20px;
  word-wrap: break-word;
  /* Ensures long words break correctly */
  white-space: normal;
  /* Ensures multi-line text */
  overflow-wrap: break-word;
  /* For better handling of long unbreakable words */
}

.bot-bubble {
  background-color: #4CAF50;
  height: fit-content;
}

.user-bubble {
  background-color: #008CBA;
  height: fit-content;
}

.typing-bubble {
  font-style: italic;
  color: #757575;
}

.user-message {
  text-align: right;
  /* Align user messages to the right */
}

.question-options {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.v-chip {
  font-weight: bold;
  cursor: pointer;
}

.chat-input {
  width: 100%;
}

.send-btn {
  width: 100%;
  height: 100%;
}
</style>
