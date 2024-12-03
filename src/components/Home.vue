<template>
  <div class="min-h-screen bg-gray-900 flex flex-col">
    <header class="bg-gray-800 shadow-lg">
      <div class="max-w-7xl mx-auto px-4 py-4">
        <h1 class="text-2xl font-bold text-gray-100">Ollama IA</h1>
      </div>
    </header>

    <main class="flex-1 w-full mx-auto px-8 py-6">
      <div class="bg-gray-800 rounded-lg shadow-xl h-[calc(100vh-7rem)] w-full">
        <div class="h-full flex flex-col">
          <div ref="chatContainer" class="flex-1 overflow-y-auto p-4">
            <div class="flex flex-col gap-4">
              <div
                v-for="(message, index) in messages"
                :key="index"
                :class="[
                  'p-4 rounded-lg max-w-[85%] shadow-md transition-all duration-200',
                  message.role === 'user'
                    ? 'bg-indigo-600 text-gray-100 self-end'
                    : 'bg-gray-700 text-gray-100 self-start',
                  index === messages.length - 1 ? 'animate-pulse-once' : '',
                ]"
              >
                <div
                  v-html="renderMarkdown(message.content)"
                  class="prose prose-invert prose-base max-w-none"
                ></div>
              </div>
              <div
                v-if="isLoading"
                class="self-start p-4 rounded-lg max-w-[85%] bg-gray-700"
              >
                <div class="flex gap-2">
                  <div
                    class="w-2 h-2 bg-indigo-400 rounded-full animate-bounce"
                    style="animation-delay: 0s"
                  ></div>
                  <div
                    class="w-2 h-2 bg-indigo-400 rounded-full animate-bounce"
                    style="animation-delay: 0.2s"
                  ></div>
                  <div
                    class="w-2 h-2 bg-indigo-400 rounded-full animate-bounce"
                    style="animation-delay: 0.4s"
                  ></div>
                </div>
              </div>
            </div>
          </div>

          <div class="border-t border-gray-700 bg-gray-800 p-4">
            <div class="max-w-7xl mx-auto flex gap-4 items-center">
              <textarea
                v-model="input"
                rows="2"
                placeholder="Ask a question"
                class="flex-1 px-4 py-2 border border-gray-600 rounded-lg bg-gray-700 text-gray-100 placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent resize-none"
                @keyup.enter.exact.prevent="ask"
              />
              <button
                @click="ask"
                :disabled="isLoading"
                class="px-6 py-2 bg-indigo-600 text-gray-100 font-medium rounded-lg hover:bg-indigo-700 transition-colors disabled:opacity-50 disabled:cursor-not-allowed"
              >
                Ask
              </button>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>
<script setup lang="ts">
import { ref, nextTick, watch, onMounted } from "vue";
import { marked } from "marked";
import hljs from "highlight.js";
import "highlight.js/styles/github-dark.css";

interface Message {
  role: "user" | "assistant";
  content: string;
}

const input = ref("");
const messages = ref<Message[]>([]);
const chatContainer = ref<HTMLElement | null>(null);
const isLoading = ref(false);

// Configuration de marked pour utiliser highlight.js
marked.setOptions({
  highlight: function (code, lang) {
    if (lang && hljs.getLanguage(lang)) {
      try {
        return hljs.highlight(code, { language: lang }).value;
      } catch (err) {
        console.error("Highlight error:", err);
      }
    }
    return hljs.highlightAuto(code).value;
  },
});

function renderMarkdown(content: string): string {
  const cleanContent = content
    .trim()
    .replace(/\n\s*\n/g, "\n")
    .replace(/^\s+/gm, "");

  return marked(cleanContent, {
    breaks: true,
    gfm: true,
  });
}

// Fonction pour appliquer highlight.js aux blocs de code après le rendu
function highlightCode() {
  nextTick(() => {
    document.querySelectorAll("pre code").forEach((block) => {
      hljs.highlightElement(block as HTMLElement);
    });
  });
}

// Observer les changements de messages pour appliquer la coloration syntaxique
watch(
  messages,
  () => {
    scrollToBottom();
    highlightCode();
  },
  { deep: true }
);

function scrollToBottom() {
  nextTick(() => {
    if (chatContainer.value) {
      chatContainer.value.scrollTop = chatContainer.value.scrollHeight;
    }
  });
}

async function ask() {
  if (!input.value.trim() || isLoading.value) return;

  const userMessage = {
    role: "user" as const,
    content: input.value.trim(),
  };

  messages.value.push(userMessage);
  input.value = ""; // Clear input immediately for better UX
  isLoading.value = true;

  try {
    const res = await fetch("http://localhost:11434/api/chat", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        model: "llama3.2",
        messages: [userMessage],
        options: {
          temperature: 0.7,
          max_tokens: 256,
        },
        stream: false,
      }),
    });

    const data = await res.json();
    const assistantMessage = {
      role: "assistant" as const,
      content: data.message.content.trim(),
    };
    messages.value.push(assistantMessage);
  } catch (error) {
    console.error("Error:", error);
    messages.value.push({
      role: "assistant",
      content: "Sorry, I encountered an error processing your request.",
    });
  } finally {
    isLoading.value = false;
  }
}
</script>
