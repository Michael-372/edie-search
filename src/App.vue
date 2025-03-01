<script setup>
import Button from "primevue/button"
import FileUpload from 'primevue/fileupload';
import ProgressSpinner from 'primevue/progressspinner';
import Panel from 'primevue/panel';

import { xml2js } from "xml-js";
import { ref,computed } from "vue";


const fileData = ref();
const totalMessages = ref(0);
const loading = ref(false);
const conversations = ref(new Map());
const selectedConversationIndex = ref();
const selectedConversation = ref();

const processData = (data) => {
  let parsed = xml2js(data,{ignoreComment: true, compact: true});
  parsed.sms.reverse()
  let grouped = groupByConversation(parsed);
  conversations.value = filterConversations(grouped);
  let output = flatten(conversations.value);
  parsed.sms = output;
  fileData.value = processData;
  totalMessages.value = parsed.sms.length;
  loading.value = false;

}

const groupByConversation = (parsedData) => {
  const map = new Map();
  for(const message of parsedData.sms) {
    let conversationId = message._attributes.address;
    if(map.has(conversationId)) {
      map.get(conversationId).push(message);
    } else {
      map.set(conversationId, [message]);
    }
  }
  return map;
}

const filterConversations = (map) => {
  let newMap = new Map();
  for(const conversationId of map.keys()) {
    const conversation = map.get(conversationId);
    if(conversation.some(c => stringHasKeyword(c._attributes.body))) {
      newMap.set(conversationId, conversation)
    }
  }
  return newMap;
}

const stringHasKeyword = (str) => {
  const keywords = [' fitzgerald',' short', ' gem',' gem land',' louise', ' brown', ' fayette', ' harkness', ' ingalls',' el destino',' righteous oaks', ' destino'];
  for (const keyword of keywords) {
    if (str.toLowerCase().includes(keyword)) {
      return true;
    }
  }
  return false;
}

const flatten = (map) => {
  let flattened = [];
  for(const conversation of map.values()) {
    flattened = flattened.concat(conversation);
  }
  return flattened;
}



const onFileSelect = (e) => {
  var reader = new FileReader();
  loading.value = true;
    reader.readAsText(e.files[0], "UTF-8");
    reader.onload = function (evt) {
      processData(evt.target.result);
    }
    reader.onerror = function (evt) {
        console.error(evt);
        loading.value = false;
    }
}
</script>

<template>
    <div style="display: flex; flex-direction: column; height: calc(100vh - 20px); margin-left: auto; margin-right: auto; max-width: 1200px; margin-bottom: 0;">
      <div style="font-size: 18px; padding: 10px;">
        XML Search: Edie Edition
      </div>
      <div v-if="!fileData" style="height: 100%; display: flex; align-items: center; justify-content: center;">
        <FileUpload v-if="!loading" mode="basic" accept="text/xml" :maxFileSize="1000000" @upload="onUpload" :auto="true" chooseLabel="Select XML File" @select="onFileSelect" />
        <ProgressSpinner v-else/>

      </div>
      <div v-else style="height: calc(100% - 52px);">
            <div style="display: flex; width: 100%; height: 100%; background-color: var(--p-surface-700); border-radius: 8px;">
              <div style="width: 180px; min-width: 180px; height: 100%;">
                <div v-for="(conversation, index) in conversations.values()" :key="conversation[0]._attributes.address" @click="selectedConversation = conversation; selectedConversationIndex = index;" class="conversation" :style="selectedConversationIndex === index ? 'background-color: var(--p-blue-500)': ''">
                   {{  conversation[0]._attributes.name || conversation[0]._attributes.address }}
                </div>
              </div>
              <div style="width: 100%; background-color: var(--p-surface-600); color: var(--p-slate-800); display: flex; flex-direction: column; gap: 12px; overflow-y: auto; padding: 12px">
                <div
                  v-for="message in selectedConversation" 
                  :class="message._attributes.type === '1' ? 'received' : 'sent'"
                >
                  <div
                    class="chat-bubble"
                    :style="message._attributes.type === '1' ? 'background-color: var(--p-slate-400)' : 'background-color: var(--p-blue-500)'"
                  >
                    {{  message._attributes.body }}
                  </div>
                  <div style="display: flex; align-items: end; font-size: 12px; color: var(--p-gray-400)">{{ message._attributes.time }}</div>
                </div>
              </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.conversation {
  background-color: var(--p-surface-800); padding: 12px; cursor: pointer;
}

.chat-bubble {
  padding: 4px;
  border-radius: 6px;
  max-width: 350px;
}
.received {
  align-self: start;
  display: flex;
  flex-direction: row;
  gap: 6px;
}

.sent {
  align-self: end;
  display: flex;
  flex-direction: row-reverse;
  gap: 6px;
}
.conversation:hover {
  opacity: 0.8
}
</style>
