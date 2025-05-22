<template>
  <div class="layout">
    <!-- Sidebar BÊN TRÁI -->
    <aside class="sidebar">
      <div class="sidebar-header">
        <h3>🍵 Góc Trà Đạo</h3>
        <p class="tagline">Hãy bắt đầu hành trình khám phá thế giới trà!</p>
      </div>

      <ul class="sidebar-menu">
        <li @click="setQuestion('Trà xanh có tác dụng gì?')">
          🌿 Trà xanh có tác dụng gì?
        </li>
        <li @click="setQuestion('Pha trà đúng cách như thế nào?')">
          🫖 Pha trà đúng cách như thế nào?
        </li>
        <li @click="setQuestion('Trà hoa nào giúp ngủ ngon?')">
          🌸 Trà hoa nào giúp ngủ ngon?
        </li>
        <li @click="setQuestion('Sự khác nhau giữa hồng trà và lục trà?')">
          🍂 Sự khác nhau giữa hồng trà và lục trà?
        </li>
      </ul>

      <div class="sidebar-footer">
        <p>“Một tách trà, một câu hỏi – mọi điều bắt đầu từ sự tò mò.”</p>
      </div>
    </aside>

    <!-- Khung chat -->
    <div class="chat-container">
      <div ref="chatBox" class="chat-box">
        <div v-for="(msg, index) in messages" :key="index" class="message-pair">
          <div class="message user">
            <strong>Your question:</strong>
            <p>{{ msg.question }}</p>
          </div>
          <div class="message ai">
            <strong>AI's answer:</strong>
            <p v-html="msg.answer"></p>
          </div>
        </div>
        <div v-if="loading" class="message ai loading">
          <strong>AI:</strong>
          <p>Đang trả lời...</p>
        </div>
        <!-- ✅ ref bên trong chat-box -->
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
      if (end && end.scrollIntoView) {
        end.scrollIntoView({
          behavior: "smooth",
          block: "end",
          inline: "nearest",
        });
      }
    },
    setQuestion(text) {
      this.input = text;
      this.$nextTick(() => {
        const textarea = document.querySelector("textarea");
        if (textarea) textarea.focus();
      });
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
  width: 240px;
  background: rgba(255, 182, 193, 0.157);
  border-right: 1px solid #ccc;
  border-left: 1px solid #ccc;
  padding: 20px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  font-family: "Georgia", serif;
  color: #2c3e50;
}

.sidebar-header h3 {
  font-size: 20px;
  margin-bottom: 6px;
  color: #3b5d3b;
}

.sidebar-header .tagline {
  font-size: 14px;
  color: #5d755d;
}

.sidebar-menu {
  list-style: none;
  padding: 0;
  margin: 20px 0;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.sidebar-menu li {
  background: #ffffff;
  padding: 10px 12px;
  border-left: 4px solid #9ac49a;
  border-radius: 6px;
  font-size: 15px;
  color: #2f4f2f;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  transition: background 0.3s;
  cursor: pointer;
}

.sidebar-menu li:hover {
  background-color: #e8f3e8;
}

.sidebar-footer {
  font-size: 13px;
  color: #4f6b4f;
  font-style: italic;
  text-align: center;
  margin-top: auto;
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
  max-height: 100%;
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
