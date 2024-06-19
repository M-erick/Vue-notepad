<template>
  <div class="flex w-screen h-screen text-gray-700">
    <div class="flex flex-col flex-shrink-0 w-64 border-r border-gray-300 bg-gray-100">

      <div class="h-0 overflow-auto flex-grow">
        <div class="flex items-center h-8 px-3 group mt-4">

          <div class="flex items-center flex-grow focus:outline-none">

            <a class="ml-2 leading-none font-medium text-sm" href="#" @click.prevent="showAllNotes">All Notes</a>

          </div>
          <button class="add-note" @click="addNewNote">
            <!-- add font awesome icon here -->
            Notes
            <svg></svg>
          </button>
        </div>

        <div class="mt-4">
          <a href="#" class="flex items-center h-8 text-sm pl-8 pr-3" v-for="note in notes" :key="note.created"
            @click.prevent="openNote(note)">
            <span class="ml-2 leading-none"> {{ new Date(note.created).toLocaleString() }}</span>

          </a>

        </div>

      </div>
    </div>
    <div class="flex flex-col flex-grow" v-if="Object.keys(activeNote).length">
      <div class="flex flex-col flex-grow overflow-auto">
        <EditorContent :editor="editor" />
      </div>
      <div class="h-16 bg-gray-100 border-t border-gray-300 text-right">
        <button @click="saveNote">Save Notes</button>
      </div>
    </div>
    <div class="flex flex-col flex-grow" v-else>
      <div v-for="note in notes" :key="note.created">
        <div class="flex px-4 pt-3 pb-4">
          <div class="prose my-2 mx-auto">
            <div>
              <span class="ml-1 text-xs text-gray-500">
                {{new Date(note.created).toLocaleString() }}

              </span>
            </div>
            <div v-html="note.content"></div>

          </div>

        </div>
        <hr class="w-full">

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
      notes:[],
      activeNote:{},
    }
  },
  async created() {
    this.database = await this.getDatabase();
    let notes = await this.getNotes();
    this.notes = notes.reverse();
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
        let noteStore = this.database.transaction('notes', 'readwrite')
          .objectStore('notes');
        let noteRequest = noteStore.get(this.activeNote.created);
        noteRequest.onerror = e => {
          reject('Error saving the note in  teh database');
        };

        noteRequest.onsuccess = e => {
          let note = e.target.result;
          note.content = this.editor.getHTML();
          let updateRequest = noteStore.put(note);


          updateRequest.onerror = e => {
            reject('Error storing the updated note in teh database ');

          };
          updateRequest.onsuccess = e => {
            let noteIndex = this.notes.findIndex(n => n.created === note.created);
            this.notes[noteIndex] = note;
            resolve();
          }
        }
      });



    },
    async getNotes(){
      return new Promise((resolve,reject)=>{
        this.database.transaction('notes')
        .objectStore('notes')
        .getAll()
        .onsuccess = e =>{
          resolve(e.target.result);
        }

      });
    },
    openNote(note){
      this.editor.commands.setContent(note.content);
      this.activeNote = note;


    },
    showAllNotes(){
      this.editor.commands.clearContent();
      this.activeNote = { };
    },
    addNewNote(){
      return new Promise((resolve,reject)=>{
        let transaction = this.database.transaction('notes','readwrite');
        transaction.oncomplete = e =>{ 
          resolve();
        }

        let now = new Date();
        let note = {
          content:'',
          created:now.getTime()
        };

        this.notes.unshift(note);
        this.activeNote = note;
        transaction.objectStore('notes').add(note);

      });
    },
  }
}
</script>

<style>
.custom-editor-style {
  @apply prose my-6 mx-auto focus:outline-none;
}
</style>
