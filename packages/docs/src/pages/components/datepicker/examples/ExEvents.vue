<template>
    <span>
        <b-field>
            <b-switch v-model="bars">Bars</b-switch>
        </b-field>
        <b-datepicker
            inline
            v-model="date"
            :events="events"
            :indicators="indicators"
            :unselectable-dates="unselectableDates"
        >
        </b-datepicker>
    </span>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import { BDatepicker, BField, BSwitch } from "buefy";

const thisMonth = new Date().getMonth();

export default defineComponent({
    components: {
        BDatepicker,
        BField,
        BSwitch,
    },
    computed: {
        indicators() {
            return this.bars ? "bars" : "dots";
        },
    },
    data() {
        return {
            date: new Date(2017, thisMonth, 1),
            events: [
                new Date(2017, thisMonth, 2),
                new Date(2017, thisMonth, 6),
                {
                    date: new Date(2017, thisMonth, 6),
                    type: "is-info",
                },
                {
                    date: new Date(2017, thisMonth, 8),
                    type: "is-danger",
                },
                {
                    date: new Date(2017, thisMonth, 10),
                    type: "is-success",
                },
                {
                    date: new Date(2017, thisMonth, 10),
                    type: "is-link",
                },
                new Date(2017, thisMonth, 12),
                {
                    date: new Date(2017, thisMonth, 12),
                    type: "is-warning",
                },
                {
                    date: new Date(2017, thisMonth, 16),
                    type: "is-danger",
                },
                new Date(2017, thisMonth, 20),
                {
                    date: new Date(2017, thisMonth, 29),
                    type: "is-success",
                },
                {
                    date: new Date(2017, thisMonth, 29),
                    type: "is-warning",
                },
                {
                    date: new Date(2017, thisMonth, 29),
                    type: "is-info",
                },
            ],
            unselectableDates: [new Date(2017, thisMonth, 29)],
            bars: false,
        };
    },
});
</script>
