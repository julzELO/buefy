<template>
    <section>
        <b-field>
            <b-radio v-model="radio" native-value="default"> Default </b-radio>
        </b-field>
        <b-field>
            <b-radio v-model="radio" native-value="info" type="is-info">
                Info
            </b-radio>
        </b-field>
        <b-field>
            <b-radio v-model="radio" native-value="success" type="is-success">
                Success
            </b-radio>
        </b-field>
        <b-field>
            <b-radio v-model="radio" native-value="danger" type="is-danger">
                Danger
            </b-radio>
        </b-field>
        <b-field>
            <b-radio v-model="radio" native-value="warning" type="is-warning">
                Warning
            </b-radio>
        </b-field>
    </section>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import { BField, BRadio } from "buefy";

export default defineComponent({
    components: {
        BField,
        BRadio,
    },
    data() {
        return {
            radio: "default",
        };
    },
});
</script>
