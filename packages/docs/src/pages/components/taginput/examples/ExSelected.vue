<template>
    <section>
        <b-field label="Enter some tags">
            <b-taginput
                v-model="tags"
                :data="filteredTags"
                autocomplete
                ref="taginput"
                icon="label"
                placeholder="Add a tag"
                @typing="getFilteredTags"
            >
                <template v-slot="props">
                    <strong>{{ props.option.id }}</strong
                    >: {{ props.option.user.first_name }}
                </template>
                <template #empty> There are no items </template>
                <template #selected="props">
                    <b-tag
                        v-for="(tag, index) in props.tags"
                        :key="index"
                        :type="getType(tag)"
                        rounded
                        :tabstop="false"
                        ellipsis
                        closable
                        @close="
                            ($refs.taginput as BTaginputInstance).removeTag(
                                index,
                                $event
                            )
                        "
                    >
                        {{ tag.user.first_name }}
                    </b-tag>
                </template>
            </b-taginput>
        </b-field>
        <pre style="max-height: 400px"><b>Tags:</b>{{ tags }}</pre>
    </section>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import data from "@/data/sample.json";
import { BField, BTag, BTaginput } from "buefy";

type BTaginputInstance = InstanceType<typeof BTaginput>;

export default defineComponent({
    components: {
        BField,
        BTag,
        BTaginput,
    },
    data() {
        return {
            filteredTags: data,
            isSelectOnly: false,
            tags: [],
        };
    },
    methods: {
        getFilteredTags(text: number | string | undefined) {
            if (text == null) {
                return;
            }
            this.filteredTags = data.filter((option) => {
                return (
                    option.user.first_name
                        .toString()
                        .toLowerCase()
                        .indexOf(text.toString().toLowerCase()) >= 0
                );
            });
        },
        getType(tag: (typeof data)[number]) {
            if (tag.id >= 1 && tag.id < 10) {
                return "is-primary";
            } else if (tag.id >= 10 && tag.id < 20) {
                return "is-danger";
            } else if (tag.id >= 20 && tag.id < 30) {
                return "is-warning";
            } else if (tag.id >= 30 && tag.id < 40) {
                return "is-success";
            } else if (tag.id >= 40 && tag.id < 50) {
                return "is-info";
            }
        },
    },
});
</script>
