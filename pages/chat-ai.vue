<template>
  <div class="layout">
    <!-- Sidebar BÊN TRÁI -->
    <aside class="sidebar">
      <h3>Menu</h3>
      <ul>
        <li><a href="#">Ngày hôm nay</a></li>
        <li><a href="#">Hôm qua</a></li>
        <li><a href="#">Tuần này</a></li>
        <li><a href="#">Tháng này</a></li>
      </ul>
    </aside>

    <!-- Khung chat -->
    <div class="chat-container">
      <div ref="chatBox" class="chat-box">
        <div v-for="(msg, index) in messages" :key="index" class="message-pair">
          <div class="message user">
            <strong>You:</strong>
            <p>{{ msg.question }}</p>
          </div>
          <div class="message ai">
            <strong>AI:</strong>
            <p>{{ msg.answer }}</p>
          </div>
        </div>
        <div v-if="loading" class="message ai loading">
          <strong>AI:</strong>
          <p>Đang trả lời...</p>
        </div>
        <!-- Phần tử đánh dấu để scroll tới -->
        <div ref="endOfChat"></div>
      </div>

      <form @submit.prevent="sendMessage" class="chat-input">
        <textarea
          v-model="input"
          placeholder="Hãy nhập câu hỏi của bạn..."
          rows="2"
          required
        ></textarea>
        <button type="submit" :disabled="loading || !input.trim()">
          {{ loading ? "Đang gửi..." : "Gửi" }}
        </button>
      </form>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      input: "",
      loading: false,
      messages: [],
    };
  },
  async mounted() {
    await this.fetchChatHistory();
    this.scrollToBottom();
  },
  methods: {
    async fetchChatHistory() {
      try {
        const res = await this.$axios.get("/chat-ai");
        this.messages = res.data.data || [];
        this.$nextTick(this.scrollToBottom);
      } catch (error) {
        console.error("Lỗi lấy lịch sử:", error);
      }
    },
    async sendMessage() {
      const question = this.input.trim();
      if (!question) return;
      this.loading = true;
      this.input = "";

      try {
        const res = await this.$axios.post("/chat-ai", { question });
        const newMsg = {
          question,
          answer: res.data.answer || "Không có phản hồi.",
        };
        this.messages.push(newMsg);
      } catch (error) {
        this.messages.push({
          question,
          answer: "Lỗi gửi câu hỏi tới AI. Vui lòng thử lại.",
        });
      } finally {
        this.loading = false;
        this.$nextTick(this.scrollToBottom);
      }
    },
    scrollToBottom() {
      const end = this.$refs.endOfChat;
      if (end) {
        end.scrollIntoView({ behavior: "smooth" });
      }
    },
  },
};
</script>
<style scoped>
.layout {
  display: flex;
  justify-content: center;
  max-width: 1200px;
  margin: auto;
  height: 80vh;
  overflow: hidden;
}

.sidebar {
  width: 200px;
  background: #fff;
  border-right: 1px solid #ddd;
  padding: 20px;
}

.chat-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: 20px;
  background: #f7f9fa;
  overflow: hidden;
}

.chat-box {
  flex: 1;
  overflow-y: auto;
  margin-bottom: 12px;
  scroll-behavior: smooth;
}

.message-pair {
  margin-bottom: 20px;
}

.message {
  background: #fff;
  padding: 12px;
  border-radius: 8px;
  margin-bottom: 6px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.05);
}

.message.user {
  background-color: #e1ffe1;
}

.message.ai {
  background-color: #e6f0ff;
}

.message.ai.loading p {
  font-style: italic;
  color: #666;
}

.chat-input {
  display: flex;
  gap: 10px;
  margin-top: auto;
}

.chat-input textarea {
  flex: 1;
  resize: none;
  padding: 10px;
  border-radius: 8px;
  font-size: 14px;
  border: 1px solid #ccc;
}

.chat-input button {
  background: #007bff;
  color: white;
  border: none;
  padding: 12px 16px;
  border-radius: 8px;
  font-weight: bold;
}
</style>
