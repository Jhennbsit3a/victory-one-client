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
export default {
  data() {
    return {
      messages: [],
      allQuestions: [
        {
          text: "What are your services?",
          response: "We provide ordering services!",
          next: ["Where are you located?", "How do I track my order?"],
        },
        {
          text: "What payment methods do you accept?",
          response: "We accept Gcash, Cash On Delivery, and Pickup.",
          next: ["Do you offer free shipping?", "Can I cancel my order?"],
        },
        {
          text: "Do you ship internationally?",
          response: "Currently, we do not offer international shipping.",
          next: ["What are your services?", "What payment methods do you accept?"],
        },
        {
          text: "Where are you located?",
          response: "We are located in San Matias, San Fernando, Pampanga.",
          next: ["What are your services?", "What payment methods do you accept?"],
        },
        {
          text: "How do I track my order?",
          response: "You can track your order in your Profile then there's a list saying 'My Order' section.",
          next: ["Do you ship internationally?", "Can I cancel my order?"],
        },
        {
          text: "Do you offer free shipping?",
          response: "Yes, we offer free shipping on orders over ₱1000.",
          next: ["What are your services?", "What payment methods do you accept?"],
        },
        {
          text: "Can I cancel my order?",
          response: "Orders can be canceled within 24 hours of purchase.",
          next: ["What are your services?", "What payment methods do you accept?"],
        },
      ],
      currentQuestions: [],
      userInput: "",
    };
  },
  created() {
    this.displayBotMessage("Hi! How can I help you today?", 2000);
    this.resetQuestions();
  },
  methods: {
    resetQuestions() {
      this.currentQuestions = this.allQuestions.slice(0, 3);
    },
    handleQuestionClick(question) {
      // Add the user's question to the chat
      this.messages.push({ type: "user", text: question.text });
      // Display typing indicator and then bot response
      this.displayBotMessage(question.response);
      // Update current questions based on the selected question's next array
      const nextQuestions = question.next.map((text) =>
        this.allQuestions.find((q) => q.text === text)
      );
      this.currentQuestions = nextQuestions.length ? nextQuestions : this.allQuestions.slice(0, 3);
    },
    handleSend() {
      if (this.userInput.trim()) {
        // Add the user's input to the chat
        const userText = this.userInput.trim();
        this.messages.push({ type: "user", text: userText });

        // Check for matching questions
        const matchedQuestion = this.allQuestions.find((question) =>
          question.text.toLowerCase().includes(userText.toLowerCase())
        );

        if (matchedQuestion) {
          // Display matched question's response
          this.displayBotMessage(matchedQuestion.response);
          const nextQuestions = matchedQuestion.next.map((text) =>
            this.allQuestions.find((q) => q.text === text)
          );
          this.currentQuestions = nextQuestions.length ? nextQuestions : this.allQuestions.slice(0, 3);
        } else {
          // Display default response
          this.displayBotMessage("I'm sorry, I didn't understand that. Could you rephrase?");
        }

        // Clear the input field
        this.userInput = "";
      }
    },
    displayBotMessage(messageText, delay = 2000) {
      this.messages.push({ type: "typing" });
      setTimeout(() => {
        // Remove typing indicator
        this.messages.pop();
        // Add bot message
        this.messages.push({ type: "bot", text: messageText });
        // Scroll to the bottom of the chat
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
}

.message-bubble {
  padding: 12px 16px;
  font-size: 14px;
  max-width: 80%;
  border-radius: 20px;
  word-wrap: break-word;
}

.bot-bubble {
  background-color: #4CAF50;
}

.user-bubble {
  background-color: #008CBA;
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
