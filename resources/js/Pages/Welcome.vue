<script setup>
import { Head, Link } from "@inertiajs/vue3";
import Echo from "laravel-echo";

defineProps({
    canLogin: Boolean,
    canRegister: Boolean,
    laravelVersion: String,
    phpVersion: String,
});
</script>

<template>
    <Head title="Welcome" />

    <div
        class="relative flex items-top justify-center min-h-screen bg-gray-100 dark:bg-gray-900 sm:items-center sm:pt-0"
    >
        <div
            v-if="canLogin"
            class="hidden fixed top-0 right-0 px-6 py-4 sm:block"
        >
            <Link
                v-if="$page.props.auth.user"
                :href="route('dashboard')"
                class="text-sm text-gray-700 dark:text-gray-500 underline"
                >Dashboard</Link
            >

            <template v-else>
                <Link
                    :href="route('login')"
                    class="text-sm text-gray-700 dark:text-gray-500 underline"
                    >Log in</Link
                >

                <Link
                    v-if="canRegister"
                    :href="route('register')"
                    class="ml-4 text-sm text-gray-700 dark:text-gray-500 underline"
                    >Register</Link
                >
            </template>
        </div>

        <div class="max-w-6xl mx-auto sm:px-6 lg:px-8">
            <div v-for="message in messages" class="text-white">
                {{ message }}
            </div>
            <div class="flex items-stretch space-x-2">
                <input
                    type="text"
                    v-model="message"
                    @keydown="tapParticipant"
                />
                <button
                    @click="sentMessage"
                    class="bg-gray-100 rounded px-4 text-sm"
                >
                    Sent
                </button>
            </div>
            <div v-if="activePeer" class="text-gray-500 text-sm mt-3">
                {{ activePeer.name }} is typing
            </div>
            <div
                v-for="participant in participants"
                class="text-gray-500 text-sm mt-3"
            >
                {{ participant.name }}
            </div>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            messages: [],
            message: "",
            activePeer: "",
            timer: false,
            participants: [],
        };
    },

    mounted() {
        window.Echo.join("private-chat")
            .here((users) => {
                this.participants = users;
                console.log(users);
            })
            .joining((user) => {
                this.participants.push(user);
                console.log("joining " + user.name);
            })
            .leaving((user) => {
                this.participants.splice(this.participants.indexOf(user), 1);
                console.log("leaving " + user.name);
            })
            .listen("MessageSent", (e) => {
                this.messages.push(e.message);
            })
            .listenForWhisper("typing", (e) => {
                this.activePeer = e;

                if (this.timer) clearTimeout(this.timer);

                this.timer = setTimeout(() => {
                    this.activePeer = false;
                }, 3000);
            });
    },

    methods: {
        sentMessage() {
            axios.get("/add-task/" + this.message);

            this.messages.push(this.message);

            this.message = "";
        },

        tapParticipant() {
            window.Echo.join("private-chat").whisper("typing", {
                name: this.$page.props.auth.user.name,
            });
        },
    },
};
</script>
