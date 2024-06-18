<template>
  <div class="flex w-screen h-screen text-gray-700">
    <div class="flex flex-col flex-shrink-0 w-64 border-r border-gray-300 bg-gray-100">
      <!-- Sidebar content -->
    </div>
    <div class="flex flex-col flex-grow">
      <div class="flex flex-col flex-grow overflow-auto">
        <EditorContent :editor="editor" />
      </div>
      <div class="h-16 bg-gray-100 border-t border-gray-300 text-right">
        <button @click="saveNote">Save Notes</button>
      </div>
    </div>
  </div>
</template>

<script>
import { Editor, EditorContent } from '@tiptap/vue-3';
import StarterKit from '@tiptap/starter-kit';

export default {
  name: 'App',
  components: {
    EditorContent
  },
  data() {
    return {
      editor: null,
      database: null,
    }
  },
  async created() {
    this.database = await this.getDatabase();
  },
  mounted() {
    this.editor = new Editor({
      content: '',
      extensions: [
        StarterKit
      ],
      editorProps: {
        attributes: {
          class: "custom-editor-style"
        }
      }
    });
  },
  beforeUnmount() {
    this.editor.destroy();
  },
  methods: {
    async getDatabase() {
      return new Promise((resolve, reject) => {
        let db = window.indexedDB.open("notes", 2);
        db.onerror = e => {
          reject("Error opening the database");
        };
        db.onsuccess = e => {
          console.log('db.onsuccess', e);
          resolve(e.target.result);
        };
        db.onupgradeneeded = e => {
          let db = e.target.result;
          if (db.objectStoreNames.contains("notes")) {
            db.deleteObjectStore("notes");
          }
          db.createObjectStore("notes", { keyPath: "created" });
        };
      });
    },
    async saveNote() {
      if (!this.database) {
        console.error("Database not initialized");
        return;
      }

      return new Promise((resolve, reject) => {
        let transaction = this.database.transaction('notes', 'readwrite');
        transaction.oncomplete = e => {
          resolve();
        };
        transaction.onerror = e => {
          reject("Error saving the note");
        };

        let now = new Date();
        let store = transaction.objectStore('notes');
        store.add({
          content: this.editor.getHTML(),
          created: now.getTime()
        });
      });
    }
  }
}
</script>

<style>
.custom-editor-style {
  @apply prose my-6 mx-auto focus:outline-none;
}
</style>
