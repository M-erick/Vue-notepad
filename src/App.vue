<template>
  <div class="flex w-screen h-screen text-gray-700">
    <div class="flex flex-col flex-shrink-0 w-64 border-r border-gray-300 bg-gray-100">
      <!-- Sidebar content -->
    </div>
    <div class="flex flex-col flex-grow">
      <div class="flex flex-col flex-grow overflow-auto">
        <EditorContent :editor="editor" />
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
      editor: null
    }
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
  }
}
</script>

<style>
.custom-editor-style {
  @apply prose my-6 mx-auto focus:outline-none;
}
</style>
