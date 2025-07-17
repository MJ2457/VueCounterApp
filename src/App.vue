<script setup>
import { ref, onMounted, watch } from "vue";

const showModel = ref(false);
const newNote = ref("");
const notes = ref([]);
const editingNote = ref(null);
const editText = ref("");

// Load notes from localStorage on component mount
onMounted(() => {
  const savedNotes = localStorage.getItem('notes');
  if (savedNotes) {
    notes.value = JSON.parse(savedNotes);
  }
});

// Watch for changes in notes array and save to localStorage
watch(notes, (newNotes) => {
  localStorage.setItem('notes', JSON.stringify(newNotes));
}, { deep: true });

function getRandomColor() {
  // Generate deeper and more saturated colors
  const hue = Math.floor(Math.random() * 360);
  const saturation = Math.floor(Math.random() * 20) + 80; // 80-100%
  const lightness = Math.floor(Math.random() * 20) + 35; // 35-55%
  return `hsl(${hue}, ${saturation}%, ${lightness}%)`;
}

const addNote = () => {
  if (newNote.value.trim() !== "") {
    notes.value.push({
      id: Math.floor(Math.random() * 1000000),
      text: newNote.value,
      date: new Date().toLocaleDateString(),
      backgroundColor: getRandomColor()
    });
    showModel.value = false;
    newNote.value = "";
  }
};

const deleteNote = (id) => {
  notes.value = notes.value.filter(note => note.id !== id);
};

const startEdit = (note) => {
  editingNote.value = note.id;
  editText.value = note.text;
};

const saveEdit = () => {
  const noteIndex = notes.value.findIndex(note => note.id === editingNote.value);
  if (noteIndex !== -1) {
    notes.value[noteIndex].text = editText.value;
    notes.value[noteIndex].date = new Date().toLocaleDateString(); // Update date when edited
  }
  editingNote.value = null;
  editText.value = "";
};

const cancelEdit = () => {
  editingNote.value = null;
  editText.value = "";
};
</script>
<template>
  <main class="min-h-screen w-screen bg-gray-100">
    <!--Modal for adding notes-->
    <div
      v-if="showModel"
      class="fixed inset-0 bg-black/80 opacity-100 z-10 flex items-center justify-center p-4"
    >
      <div class="w-full max-w-md bg-white rounded-xl p-6 relative flex flex-col">
        <textarea 
          v-model="newNote" 
          name="note" 
          id="note"
          placeholder="Enter your note..."
          spellcheck="true"
          lang="en"
          class="w-full h-32 p-3 border border-gray-300 rounded-lg resize-none focus:outline-none focus:ring-2 focus:ring-violet-500 focus:border-transparent"
        ></textarea>
        <button @click="addNote"
          class="p-3 text-lg w-full bg-violet-700 text-white cursor-pointer mt-4 rounded-lg hover:bg-violet-800 transition-colors duration-200 ease-in-out"
        >
          Add Note
        </button>
        <button
          @click="showModel = false"
          class="bg-red-700 mt-3 p-3 text-white rounded-lg hover:bg-red-800 transition-colors duration-200 ease-in-out"
        >
          Close
        </button>
      </div>
    </div>
    <div class="w-full max-w-6xl mx-auto p-4 sm:p-6 lg:p-8">
      <header class="flex flex-col sm:flex-row justify-between items-center gap-4 sm:gap-0 p-4 sm:p-6 lg:p-8">
        <h1 class="text-4xl sm:text-5xl lg:text-7xl font-bold text-gray-800 text-center sm:text-left">Notes</h1>
        <button
          @click="showModel = true"
          class="bg-blue-500 hover:bg-blue-600 text-white text-xl sm:text-2xl font-bold py-3 px-4 sm:py-2 sm:px-4 rounded-full shadow-md transition-colors duration-200 ease-in-out flex items-center justify-center min-w-[50px] min-h-[50px]"
        >
          +
        </button>
      </header>
      <div class="grid grid-cols-1 xs:grid-cols-2 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5 gap-4 mt-6">
        <div
          v-for="note in notes"
          :key="note.id"
          class="w-full aspect-square max-w-48 mx-auto shadow-lg rounded-2xl p-3 flex flex-col justify-between relative group"
          :style="{ backgroundColor: note.backgroundColor }"
        >
          <!-- Note content -->
          <div v-if="editingNote === note.id" class="flex flex-col h-full">
            <textarea
              v-model="editText"
              spellcheck="true"
              lang="en"
              class="flex-grow resize-none border-none outline-none bg-transparent text-sm p-1 rounded focus:ring-2 focus:ring-white/50"
              @keydown.enter.ctrl="saveEdit"
              @keydown.escape="cancelEdit"
              placeholder="Edit your note..."
            ></textarea>
            <div class="flex gap-1 mt-2">
              <button
                @click="saveEdit"
                class="bg-green-600 hover:bg-green-700 text-white text-xs px-2 py-1 rounded flex-1 transition-colors duration-200"
              >
                Save
              </button>
              <button
                @click="cancelEdit"
                class="bg-gray-600 hover:bg-gray-700 text-white text-xs px-2 py-1 rounded flex-1 transition-colors duration-200"
              >
                Cancel
              </button>
            </div>
          </div>
          
          <div v-else class="flex flex-col h-full">
            <p class="main-text text-sm flex-grow leading-relaxed text-white/90">{{ note.text }}</p>
            <div class="flex justify-between items-center mt-auto pt-2">
              <p class="font-sm font-bold text-xs text-white/80">{{ note.date }}</p>
              <!-- Edit/Delete buttons (show on hover or always on mobile) -->
              <div class="opacity-100 sm:opacity-0 sm:group-hover:opacity-100 transition-opacity duration-200 flex gap-1">
                <button
                  @click="startEdit(note)"
                  class="bg-blue-600 hover:bg-blue-700 text-white text-xs px-2 py-1 rounded shadow-sm transition-colors duration-200"
                >
                  ✏️
                </button>
                <button
                  @click="deleteNote(note.id)"
                  class="bg-red-600 hover:bg-red-700 text-white text-xs px-2 py-1 rounded shadow-sm transition-colors duration-200"
                >
                  🗑️
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>
