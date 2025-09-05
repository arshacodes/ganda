<script setup>
import { formatDistanceToNow } from "date-fns";
import { db, auth } from "@/config/firebaseConfig";
import { doc, deleteDoc, updateDoc } from "firebase/firestore";
import { onAuthStateChanged } from "firebase/auth";
import { ref, onMounted } from "vue";
import { toast } from "vue3-toastify";
import { parse, format } from 'date-fns';

const formatTime = (time24) => {
  const parsedTime = parse(time24, 'HH:mm', new Date());
  return format(parsedTime, 'h:mm a');
};

const props = defineProps({
    post: { type: Object, required: true },
});

const emit = defineEmits(["deleted", "edit"]);

const currentUser = ref(null);
const isSpeedDialOpen = ref(false);


const confirmModalVisible = ref(false);
const eventToDelete = ref(null);


const isEditModalOpen = ref(false);
const editingEvent = ref(null);
const isSubmitting = ref(false);

onMounted(() => {
    onAuthStateChanged(auth, (user) => {
        currentUser.value = user;
    });
});

// Show confirmation modal
const askDeleteConfirmation = (eventId) => {
    eventToDelete.value = eventId;
    confirmModalVisible.value = true;
};

// Delete event
const deleteEvent = async () => {
    if (!eventToDelete.value) return;
    try {
        await deleteDoc(doc(db, "events", eventToDelete.value));
        toast.success("Event deleted successfully");
        emit("deleted", eventToDelete.value);
    } catch (error) {
        console.error("Error deleting event: ", error);
        toast.error("Failed to delete event");
    } finally {
        confirmModalVisible.value = false;
        eventToDelete.value = null;
    }
};

// Open edit modal
const openEditModal = (post) => {
  editingEvent.value = { ...post }; // shallow copy for v-model
  isEditModalOpen.value = true;
};

// Close edit modal
const closeEditModal = () => {
  isEditModalOpen.value = false;
  editingEvent.value = null;
};

// Update event
const updateEvent = async () => {
  if (!editingEvent.value?.id) return;

  try {
    isSubmitting.value = true;
    await updateDoc(doc(db, "events", editingEvent.value.id), {
      title: editingEvent.value.title,
      description: editingEvent.value.description,
      location: editingEvent.value.location,
      date: editingEvent.value.date,
      startTime: editingEvent.value.startTime,
      endTime: editingEvent.value.endTime,
    });
    toast.success("Event updated successfully");
    closeEditModal();
  } catch (error) {
    console.error("Error updating event:", error);
    toast.error("Failed to update event");
  } finally {
    isSubmitting.value = false;
  }
};
</script>

<template>
    <div class="bg-white rounded-lg shadow overflow-hidden w-full relative">
        <div class="flex flex-col md:flex-row">
        <img
            :src="post?.eventImage || '/src/components/ui/events/default.png'"
            alt="Event image"
            class="w-full md:w-1/3 h-64 md:h-auto object-cover"
        />

        <div class="flex-1 p-4 flex flex-col justify-between">
            <div class="flex items-start justify-between">
                <div>
                    <div class="flex items-center space-x-2 mb-2">
                        <img
                            :src="post?.user?.photoUrl || '/default-avatar.png'"
                            alt="User avatar"
                            class="w-8 h-8 rounded-full object-cover"
                        />
                        <h4 @click="$emit('visitProfile', post.uid)" class="font-semibold text-gray-800 cursor-pointer hover:underline">
                            {{
                            post?.user?.firstName || post?.user?.lastName
                                ? (post?.user?.firstName || '') + ' ' + (post?.user?.lastName || '')
                                : post?.user?.email || 'Unknown User'
                            }}
                        </h4>
                    </div>
                    <span class="text-sm text-gray-500">
                        Posted {{
                            formatDistanceToNow(post.createdAt?.toDate?.() || new Date(), {
                            addSuffix: true,
                            })
                        }}
                    </span>
                </div>

                <div v-if="currentUser?.uid === post?.uid" class="relative flex items-end">
                    <button
                    @click="isSpeedDialOpen = !isSpeedDialOpen"
                    class="w-10 h-10 flex items-center justify-center bg-indigo-600 text-white rounded-full shadow-md hover:bg-indigo-700 focus:outline-none"
                    >
                        <svg
                            v-if="!isSpeedDialOpen"
                            xmlns="http://www.w3.org/2000/svg"
                            class="w-5 h-5"
                            fill="none"
                            viewBox="0 0 24 24"
                            stroke="currentColor"
                            stroke-width="2"
                        >
                            <path stroke-linecap="round" stroke-linejoin="round" d="M12 4v16m8-8H4" />
                        </svg>
                        <svg
                            v-else
                            xmlns="http://www.w3.org/2000/svg"
                            class="w-5 h-5"
                            fill="none"
                            viewBox="0 0 24 24"
                            stroke="currentColor"
                            stroke-width="2"
                        >
                            <path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
                        </svg>
                    </button>

                    <transition name="fade">
                        <div
                            v-if="isSpeedDialOpen"
                            class="absolute top-12 right-0 flex flex-col items-center space-y-2"
                        >

                            <button
                            @click="openEditModal(post)"
                            class="w-9 h-9 flex items-center justify-center bg-blue-500 text-white rounded-full shadow hover:bg-blue-600"
                            title="Edit Event"
                            >
                                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
                                    <path fill="currentColor" d="m18.988 2.012l3 3L19.701 7.3l-3-3zM8 16h3l7.287-7.287l-3-3L8 13z"/>
                                    <path fill="currentColor" d="M19 19H8.158c-.026 0-.053.01-.079.01c-.033 0-.066-.009-.1-.01H5V5h6.847l2-2H5c-1.103 0-2 .896-2 2v14c0 1.104.897 2 2 2h14a2 2 0 0 0 2-2v-8.668l-2 2z"/>
                                </svg>
                            </button>

                            <button
                            @click="askDeleteConfirmation(post.id)"
                            class="w-9 h-9 flex items-center justify-center bg-red-500 text-white rounded-full shadow hover:bg-red-600"
                            title="Delete Event"
                            >
                                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
                                    <path fill="currentColor" d="M7 21q-.825 0-1.412-.587T5 19V6H4V4h5V3h6v1h5v2h-1v13q0 .825-.587 1.413T17 21zm2-4h2V8H9zm4 0h2V8h-2z"/>
                                </svg>
                            </button>
                        </div>
                    </transition>
                </div>
            </div>

            <!-- Event Details -->
            <div class="mt-4">
                <h2 class="text-xl font-bold text-indigo-700">{{ post.title }}</h2>
                <p class="text-gray-700 mt-2 whitespace-pre-line">{{ post.description }}</p>
                <div class="mt-3 text-sm text-gray-600 space-y-1">
                    <div><strong>Location:</strong> {{ post.location }}</div>
                    <div><strong>Date:</strong> {{ post.date }}</div>
                    <div><strong>Time:</strong> {{ formatTime(post.startTime) }} – {{ formatTime(post.endTime) }}</div>
                </div>
            </div>
        </div>
        </div>

        <!-- Delete Confirmation Modal -->
        <transition name="fade">
        <div
            v-if="confirmModalVisible"
            class="fixed inset-0 flex items-center justify-center bg-black/40 backdrop-blur-sm z-50"
        >
            <div class="bg-white rounded-lg shadow-lg p-6 w-80 max-w-full">
                <h3 class="text-lg font-bold mb-4">Delete Event?</h3>
                <p class="mb-6">Are you sure you want to delete this event?</p>
                <div class="flex justify-end space-x-2">
                    <button
                    @click="confirmModalVisible = false"
                    class="px-4 py-2 bg-gray-300 rounded hover:bg-gray-400"
                    >
                    Cancel
                    </button>
                    
                    <button
                    @click="deleteEvent"
                    class="px-4 py-2 bg-red-500 text-white rounded hover:bg-red-600"
                    >
                    Confirm
                    </button>
            </div>
            </div>
        </div>
        </transition>

        <!-- Edit Modal -->
        <transition name="fade">
        <div
            v-if="isEditModalOpen"
            class="fixed inset-0 flex items-center justify-center bg-black/40 backdrop-blur-sm z-50"
        >
            <div class="bg-white rounded-lg shadow-lg p-6 w-96 max-w-full">
            <h2 class="text-xl font-semibold mb-4">Edit Event</h2>

            <form @submit.prevent="updateEvent" class="space-y-4">
                <div>
                <label class="block font-medium mb-1" for="title">Title *</label>
                <input
                    id="title"
                    v-model="editingEvent.title"
                    type="text"
                    class="w-full border rounded px-3 py-2 focus:outline-none focus:ring-2 focus:ring-indigo-500"
                    required
                />
                </div>

                <div>
                <label class="block font-medium mb-1" for="description">Description *</label>
                <textarea
                    id="description"
                    v-model="editingEvent.description"
                    rows="3"
                    class="w-full border rounded px-3 py-2 focus:outline-none focus:ring-2 focus:ring-indigo-500"
                    required
                ></textarea>
                </div>

                <div>
                <label class="block font-medium mb-1" for="location">Location</label>
                <input
                    id="location"
                    v-model="editingEvent.location"
                    type="text"
                    class="w-full border rounded px-3 py-2 focus:outline-none focus:ring-2 focus:ring-indigo-500"
                />
                </div>

                <div>
                <label class="block font-medium mb-1" for="date">Date *</label>
                <input
                    id="date"
                    v-model="editingEvent.date"
                    type="date"
                    class="w-full border rounded px-3 py-2 focus:outline-none focus:ring-2 focus:ring-indigo-500"
                    required
                />
                </div>

                <div class="flex space-x-4">
                <div class="flex-1">
                    <label class="block font-medium mb-1" for="startTime">Start Time *</label>
                    <input
                    id="startTime"
                    v-model="editingEvent.startTime"
                    type="time"
                    class="w-full border rounded px-3 py-2 focus:outline-none focus:ring-2 focus:ring-indigo-500"
                    required
                    />
                </div>
                <div class="flex-1">
                    <label class="block font-medium mb-1" for="endTime">End Time *</label>
                    <input
                    id="endTime"
                    v-model="editingEvent.endTime"
                    type="time"
                    class="w-full border rounded px-3 py-2 focus:outline-none focus:ring-2 focus:ring-indigo-500"
                    required
                    />
                </div>
                </div>

                <div class="flex justify-end space-x-2 mt-4">
                <button
                    type="button"
                    @click="closeEditModal"
                    class="px-4 py-2 bg-gray-300 rounded hover:bg-gray-400"
                >
                    Cancel
                </button>
                <button
                    type="submit"
                    :disabled="isSubmitting"
                    class="px-4 py-2 bg-indigo-600 text-white rounded hover:bg-indigo-700 disabled:bg-indigo-400"
                >
                    {{ isSubmitting ? 'Updating...' : 'Update Event' }}
                </button>
                </div>
            </form>
            </div>
        </div>
        </transition>
    </div>
</template>

<style scoped>
.fade-enter-active, .fade-leave-active {
    transition: all 0.2s ease;
}
.fade-enter-from, .fade-leave-to {
    opacity: 0;
    transform: translateY(-5px);
}
</style>