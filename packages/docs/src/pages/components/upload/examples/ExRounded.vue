<template>
    <section>
        <b-field label="Included filename">
            <b-field class="file is-primary" :class="{ 'has-name': !!file }">
                <b-upload v-model="file" class="file-label" rounded>
                    <span class="file-cta">
                        <b-icon class="file-icon" icon="upload"></b-icon>
                        <span class="file-label">{{
                            file.name || "Click to upload"
                        }}</span>
                    </span>
                </b-upload>
            </b-field>
        </b-field>

        <b-field label="Separated filename">
            <b-field class="file is-primary" :class="{ 'has-name': !!file2 }">
                <b-upload v-model="file2" class="file-label" rounded>
                    <span class="file-cta">
                        <b-icon class="file-icon" icon="upload"></b-icon>
                        <span class="file-label">Click to upload</span>
                    </span>
                    <span class="file-name" v-if="file2">
                        {{ file2.name }}
                    </span>
                </b-upload>
            </b-field>
        </b-field>
    </section>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import { BField, BIcon, BUpload } from "buefy";

export default defineComponent({
    components: {
        BField,
        BIcon,
        BUpload,
    },
    data() {
        return {
            file: {} as File,
            file2: null as File | null,
        };
    },
});
</script>
