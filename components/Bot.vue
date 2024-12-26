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
      <v-col cols="12">
        <v-row justify="center" align="center">
          <v-col cols="9" class="pr-1">
            <v-text-field v-model="userInput" placeholder="Type your message..." outlined dense
              class="chat-input"></v-text-field>
          </v-col>
          <v-col cols="3" class="pl-1">
            <v-btn color="primary" @click="handleSend" elevation="2" class="send-btn" block>
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
          response: "We provide seamless ordering services, including a variety of delivery and pickup options for your convenience.",
          next: ["Where are you located?", "How do I track my order?"],
        },
        {
          text: "What products do you offer?",
          response: "We offer seamless steel pipes, precision tubes, alloy pipes, copper tubes, aluminum tubes, and special-shaped pipes. Our products cater to industries like chemical processing, construction, hydraulic systems, oil and gas, and more.",
          next: ["What materials are your products made from?", "What are the applications of your products?"],
        },
        {
          text: "What materials are your products made from?",
          response: "We use high-quality raw materials such as steel, copper, and aluminum in the production of our products. Our materials are carefully selected to ensure durability and efficiency across various industries.",
          next: ["What products do you offer?", "What are the applications of your products?"],
        },
        {
          text: "What are the applications of your products?",
          response: "Our products are used in industries such as fluid transport, chemical processing, boiler systems, heat exchangers, construction, and oil pipelines. They are designed for high performance and reliability in demanding applications.",
          next: ["What products do you offer?", "What materials are your products made from?"],
        },
        {
          text: "Where are you located?",
          response: "Our main location is in San Matias, San Fernando, Pampanga. Feel free to visit us during business hours!",
          next: ["What are your services?", "What payment methods do you accept?"],
        },
        {
          text: "Do you ship internationally?",
          response: "Currently, we focus on delivering within certain regions, but we are expanding our reach. For specific shipping inquiries, please contact us directly.",
          next: ["Where are you located?", "How long does shipping take?"],
        },
        {
          text: "How long does shipping take?",
          response: "Shipping times vary depending on the destination, but typically range from 7 to 14 business days for most locations. For more detailed shipping information, please contact our customer service team.",
          next: ["Do you offer free shipping?", "Can I track my order?"],
        },
        {
          text: "How do I track my order?",
          response: "To track your order, simply go to the 'My Orders' section in your Profile. You'll find all updates and delivery information there.",
          next: ["Do you ship internationally?", "Can I cancel my order?"],
        },
        {
          text: "Do you offer free shipping?",
          response: "We offer free shipping on orders over a certain amount. Please check our website or contact customer service for the latest shipping policies.",
          next: ["How long does shipping take?", "Can I track my order?"],
        },
        {
          text: "What payment methods do you accept?",
          response: "We accept various payment methods, including bank transfer, Gcash, cash on delivery (COD) and Pickup. If you have any questions about payment options, feel free to contact us.",
          next: ["How long does shipping take?", "Can I cancel my order?"],
        },
        {
          text: "Can I cancel my order?",
          response: "Orders can be canceled within 24 hours of purchase. Please contact us as soon as possible if you wish to cancel or modify your order.",
          next: ["What payment methods do you accept?", "How do I track my order?"],
        },
        {
          text: "What is your mission?",
          response: "Our mission is to deliver high-quality aluminum products and solutions that exceed customer expectations. We focus on sustainable practices, responsible sourcing, and continuous improvement.",
          next: ["What is your vision?", "What industries do you serve?"],
        },
        {
          text: "What is your vision?",
          response: "Our vision is to be the leading provider of aluminum solutions, recognized for innovation and sustainability. We aim to contribute to a greener future through our products and services.",
          next: ["What is your mission?", "What industries do you serve?"],
        },
        {
          text: "What industries do you serve?",
          response: "We serve a variety of industries, including construction, chemical processing, hydraulic systems, oil and gas, automotive, and more. Our products are designed for diverse applications in these sectors.",
          next: ["What is your mission?", "What products do you offer?"],
        },
        {
          text: "How long have you been in business?",
          response: "We have been in business for several years, focusing on high-quality production and customer satisfaction. Over time, we have grown and expanded our capabilities to serve a global market.",
          next: ["What industries do you serve?", "Can I work with you?"],
        },
        {
          text: "Can I work with you?",
          response: "We are always looking for talented individuals to join our team. Please visit our careers page for current job openings.",
          next: ["How can I contact customer support?", "How long have you been in business?"],
        },
        {
          text: "How can I contact customer support?",
          response: "You can reach our customer support team via email at support@victoryone.com or call us at 123-456-7890. We are happy to assist you with any inquiries.",
          next: ["What is your refund policy?", "What is your return policy?"],
        },
        {
          text: "Do you have an FAQ page?",
          response: "Yes, we have an FAQ page on our website that answers common questions. Please visit the FAQ section for more details.",
          next: ["How can I contact customer support?", "What is your refund policy?"],
        },
        {
          text: "What is your refund policy?",
          response: "We offer refunds for products returned in their original condition within 30 days of purchase. Please contact customer support for further details.",
          next: ["How can I contact customer support?", "Do you have an FAQ page?"],
        },
      ],
      currentQuestions: [],
      userInput: "",
      fuse: null,
    };
  },
  created() {
    this.displayBotMessage("Hi! How can I help you today?", 1000);
    this.resetQuestions();

    // Initialize Fuse.js for fuzzy matching
    this.fuse = new Fuse(this.allQuestions, {
      keys: ["text"], // Ensure the entire question text is analyzed
      threshold: 0.4, // Set a moderate threshold for meaningful matches
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
      ).filter(Boolean); // Ensure no undefined values are present

      this.currentQuestions = nextQuestions.length ? nextQuestions : this.allQuestions.slice(0, 3);
    },
    handleSend() {
      if (this.userInput.trim()) {
        const userText = this.userInput.trim().toLowerCase(); // Use the entire input for matching
        this.messages.push({ type: "user", text: this.userInput });

        // Use Fuse.js to search for the best match
        const result = this.fuse.search(userText);

        if (result.length > 0) {
          const matchedQuestion = result[0].item;
          this.displayBotMessage(matchedQuestion.response);

          const nextQuestions = matchedQuestion.next.map((text) =>
            this.allQuestions.find((q) => q.text === text)
          ).filter(Boolean); // Ensure no undefined values

          this.currentQuestions = nextQuestions.length ? nextQuestions : this.allQuestions.slice(0, 3);
        } else {
          // Friendly fallback for unmatched inputs
          this.displayBotMessage(
            "That's an interesting question! It's not available in the system right now, but feel free to ask anything else, and I'll do my best to assist you."
          );
        }

        this.userInput = ""; // Clear user input field
      }
    },
    displayBotMessage(messageText, delay = 2000) {
      this.messages.push({ type: "typing" });
      setTimeout(() => {
        this.messages.pop();
        this.messages.push({ type: "bot", text: messageText });
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
  padding-top: 24px;
}

.send-btn {
  width: 100%;
}
</style>
