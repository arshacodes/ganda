<script setup>
import { ref, onMounted } from 'vue'
import { auth, db } from '@/config/firebaseConfig'
import { createUserWithEmailAndPassword, onAuthStateChanged } from 'firebase/auth'
import { doc, setDoc, serverTimestamp } from 'firebase/firestore'
import { toast } from 'vue3-toastify'

const title = ref('')
const description = ref('')
const location = ref('')
const date = ref('')
const time = ref('')
const isSubmitting = ref(false)

const isAddEventShown = ref(false)

const toggleCreateEvent = () => {
    isAddEventShown.value = !isAddEventShown.value
    console.log(isAddEventShown.value)
}

const addEvent = async () => {
    try {
        // Save extra fields to firestore
        await setDoc(doc(db, 'events'), {
            title: title.value,
            description: description.value,
            location: location.value,
            time: time.value,
            createdAt: serverTimestamp(),
        })

        toast.success('Event created successfully')
    } catch (err) {
        console.log(err.message)
        toast.error('Failed to create event, please try again')
    } finally {
        isSubmitting.value = false
    }
}
</script>
<template>
    <div class="flex flex-row justify-between">
        <h1 class="text-2xl font-bold mb-4">Explore Events</h1>
        <button @click="toggleCreateEvent" class="bg-indigo-600 text-white px-3 py-1 rounded-md text-sm hover:bg-indigo-700">Add Event</button>
    </div>

    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        <!-- Community card template repeated for multiple communities -->
        <div class="bg-white rounded-lg shadow p-5 flex flex-col">
            <div class="flex items-center space-x-3 mb-3">
                <span
                    class="w-10 h-10 bg-indigo-100 text-indigo-600 rounded-full flex items-center justify-center font-semibold"
                    >CS</span
                >
                <div>
                    <h3 class="font-semibold text-gray-800">Computer Science Society</h3>
                    <p class="text-xs text-gray-500">1.2K members</p>
                </div>
            </div>
            <p class="text-sm text-gray-600 flex-1">
                A community for computer science enthusiasts to discuss coding, algorithms, and
                emerging technologies. Share resources, ask questions, and collaborate on projects.
            </p>
            <div class="mt-4 flex justify-between items-center">
                <a href="#" class="text-indigo-600 hover:underline text-sm">View</a>
                <button
                    class="bg-indigo-600 text-white px-3 py-1 rounded-md text-sm hover:bg-indigo-700"
                >
                    Join
                </button>
            </div>
        </div>
    </div>

    <div class="w-full mx-auto space-y-4">
        <EventCard v-for="(event, index) in events" :key="index" :event="event" />

        <!-- Loader -->
        <div v-if="loading" class="text-center py-4">
            <span>Loading...</span>
        </div>

        <!-- Trigger for infinite scroll -->
        <div ref="loadMoreTrigger" class="h-2"></div>
    </div>

    <div
        class="fixed inset-0 flex items-center justify-center bg-black/50 backdrop-blur-sm z-51"
        :class="{ hidden: !isAddEventShown }"
    >
        <div class="bg-white rounded-2xl shadow-lg w-full max-w-md p-6 relative">
            <!-- Close Button -->
            <button
                @click="isAddEventShown = false"
                class="absolute top-3 right-3 text-gray-500 hover:text-gray-700"
            >
                ✕
            </button>

            <!-- Modal Header -->
            <h2 class="text-xl font-semibold mb-4">Event Details</h2>

            <!-- Modal Form -->
            <form action="#" class="space-y-4">
                <div>
                    <label class="block text-sm font-medium text-gray-700">Title</label>
                    <input
                        v-model="title"
                        type="text"
                        class="w-full mt-1 px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none"
                    />
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-700">Description</label>
                    <textarea
                        v-model="description"
                        type="text"
                        class="w-full mt-1 px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none"
                    ></textarea>
                </div>
                <div>
                    <h2 class="text-lg font-semibold mb-2">Location</h2>
                    <input
                        v-model="location"
                        type="text"
                        class="w-full mt-1 px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none"
                    />
                </div>

                <div class="flex flex-row space-x-3 w-full">
                    <div class="w-full">
                        <label class="block text-sm font-medium text-gray-700">Date</label>
                        <input
                            v-model="date"
                            type="date"
                            class="w-full mt-1 px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none"
                        />
                    </div>

                    <div class="w-full">
                        <label class="block text-sm font-medium text-gray-700">Time</label>
                        <input
                            v-model="time"
                            type="time"
                            class="w-full mt-1 px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none"
                        />
                    </div>
                </div>

                <button
                        v-if="!isSubmitting"
                        @click="addEvent"
                        type="button"
                        class="group relative w-full flex justify-center py-2 px-4 border border-transparent text-sm font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
                    >
                        Create Event
                    </button>
                    <button
                        v-if="isSubmitting"
                        type="button"
                        class="animate-pulse group relative w-full flex justify-center py-2 px-4 border border-transparent text-sm font-medium rounded-md text-white bg-indigo-400 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
                    >
                        Creating Event...
                    </button>
            </form>
        </div>
    </div>
</template>
