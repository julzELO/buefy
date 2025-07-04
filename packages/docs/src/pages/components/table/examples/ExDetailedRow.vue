<template>
    <section>
        <b-field grouped group-multiline>
            <div class="control">
                <b-switch v-model="showDetailIcon">Show detail icon</b-switch>
            </div>
            <div class="control">
                <b-switch v-model="useTransition"
                    >Use transition (fade) when toggling details</b-switch
                >
            </div>
        </b-field>

        <b-table
            :data="data"
            ref="table"
            paginated
            per-page="5"
            :opened-detailed="defaultOpenedDetails"
            detailed
            detail-key="id"
            :detail-transition="transitionName"
            @details-open="
                (row) => $buefy.toast.open(`Expanded ${row.user.first_name}`)
            "
            :show-detail-icon="showDetailIcon"
            aria-next-label="Next page"
            aria-previous-label="Previous page"
            aria-page-label="Page"
            aria-current-label="Current page"
        >
            <b-table-column
                field="id"
                label="ID"
                width="40"
                numeric
                v-slot="props"
            >
                {{ props.row.id }}
            </b-table-column>

            <b-table-column
                field="user.first_name"
                label="First Name"
                sortable
                v-slot="props"
            >
                <template v-if="showDetailIcon">
                    {{ props.row.user.first_name }}
                </template>
                <template v-else>
                    <a @click="props.toggleDetails(props.row)">
                        {{ props.row.user.first_name }}
                    </a>
                </template>
            </b-table-column>

            <b-table-column
                field="user.last_name"
                label="Last Name"
                sortable
                v-slot="props"
            >
                {{ props.row.user.last_name }}
            </b-table-column>

            <b-table-column
                field="date"
                label="Date"
                sortable
                centered
                v-slot="props"
            >
                <span class="tag is-success">
                    {{ new Date(props.row.date).toLocaleDateString() }}
                </span>
            </b-table-column>

            <b-table-column label="Gender" v-slot="props">
                <span>
                    <b-icon
                        pack="fas"
                        :icon="props.row.gender === 'Male' ? 'mars' : 'venus'"
                    >
                    </b-icon>
                    {{ props.row.gender }}
                </span>
            </b-table-column>

            <template #detail="props">
                <article class="media">
                    <figure class="media-left">
                        <p class="image is-64x64">
                            <img src="/static/img/placeholder-128x128.png" />
                        </p>
                    </figure>
                    <div class="media-content">
                        <div class="content">
                            <p>
                                <strong
                                    >{{ props.row.user.first_name }}
                                    {{ props.row.user.last_name }}</strong
                                >
                                <small>@{{ props.row.user.first_name }}</small>
                                <small>31m</small>
                                <br />
                                Lorem ipsum dolor sit amet, consectetur
                                adipiscing elit. Proin ornare magna eros, eu
                                pellentesque tortor vestibulum ut. Maecenas non
                                massa sem. Etiam finibus odio quis feugiat
                                facilisis.
                            </p>
                        </div>
                    </div>
                </article>
            </template>
        </b-table>
    </section>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import { BField, BIcon, BSwitch, BTable, BTableColumn } from "buefy";
import data from "@/data/sample.json";

export default defineComponent({
    components: {
        BField,
        BIcon,
        BSwitch,
        BTable,
        BTableColumn,
    },
    data() {
        return {
            data,
            defaultOpenedDetails: [1],
            showDetailIcon: true,
            useTransition: false,
        };
    },
    computed: {
        transitionName() {
            if (this.useTransition) {
                return "fade";
            }
        },
    },
});
</script>
