<template>
    <section>
        <div class="buttons">
            <b-button
                label="Launch prompt (default)"
                type="is-dark"
                size="is-medium"
                @click="prompt"
            />

            <b-button
                label="Launch prompt (number)"
                type="is-dark"
                size="is-medium"
                @click="promptNumber"
            />

            <b-button
                label="Launch prompt (Not closing default)"
                type="is-dark"
                size="is-medium"
                @click="promptNotClosed"
            />

            <b-button
                label="Launch prompt (Not closing with loading)"
                type="is-dark"
                size="is-medium"
                @click="promptNotClosedWithLoading"
            />
        </div>
    </section>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import { BButton } from "buefy";

export default defineComponent({
    components: { BButton },
    methods: {
        prompt() {
            this.$buefy.dialog.prompt({
                message: `What's your name?`,
                inputAttrs: {
                    placeholder: "e.g. Walter",
                    maxlength: 10,
                },
                trapFocus: true,
                onConfirm: (value) =>
                    this.$buefy.toast.open(`Your name is: ${value}`),
            });
        },
        promptNumber() {
            this.$buefy.dialog.prompt({
                message: `What's your age?`,
                inputAttrs: {
                    type: "number",
                    placeholder: "Type your age",
                    value: "18",
                    min: 18,
                    max: 99,
                },
                trapFocus: true,
                onConfirm: (value) =>
                    this.$buefy.toast.open(`Your age is: ${value}`),
            });
        },
        promptNotClosed() {
            this.$buefy.dialog.prompt({
                message: "Send your message to moon 🚀",
                inputAttrs: {
                    type: "text",
                    placeholder: "My message is...",
                    value: "Hello moon!",
                },
                confirmText: "Send",
                trapFocus: true,
                closeOnConfirm: false,
                onConfirm: (value, { close, startLoading }) => {
                    startLoading();
                    this.$buefy.toast.open(`Your message is sending...`);
                    setTimeout(() => {
                        this.$buefy.toast.open(`Success message send!`);
                        close();
                    }, 2000);
                },
            });
        },
        promptNotClosedWithLoading() {
            this.$buefy.dialog.prompt({
                message: "Send your message to moon 🚀",
                inputAttrs: {
                    type: "text",
                    placeholder: "My message is...",
                    value: "Hello moon!",
                },
                confirmText: "Send",
                trapFocus: true,
                closeOnConfirm: false,
                onConfirm: (value, { startLoading, cancelLoading }) => {
                    startLoading();
                    this.$buefy.toast.open(`Your message is sending...`);
                    setTimeout(() => {
                        this.$buefy.toast.open(`Success message send!`);
                        cancelLoading();
                    }, 2000);
                },
            });
        },
    },
});
</script>
