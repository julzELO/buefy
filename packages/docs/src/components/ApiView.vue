<template>
    <section id="api-view" class="api-view">
        <h2 class="title is-4">
            <router-link to="#api-view">
                #
            </router-link>
            API
        </h2>

        <template v-for="component in data" :key="component.title">
            <div>
                <h3 v-if="component.title" class="subtitle">
                    {{ component.title }}
                </h3>
                <b-tabs>
                    <b-tab-item v-if="component.props" label="Properties">
                        <b-table
                            :mobile-cards="false"
                            :data="component.props"
                            :columns="propsColumns"
                        />
                    </b-tab-item>

                    <b-tab-item v-if="component.slots" label="Slots">
                        <b-table
                            :mobile-cards="false"
                            :data="component.slots"
                            :columns="slotsColumns"
                        />
                    </b-tab-item>

                    <b-tab-item v-if="component.events" label="Events">
                        <b-table
                            :mobile-cards="false"
                            :data="component.events"
                            :columns="eventsColumns"
                        />
                    </b-tab-item>

                    <b-tab-item v-if="component.methods" label="Methods">
                        <b-table
                            :mobile-cards="false"
                            :data="component.methods"
                            :columns="methodsColumns"
                        />
                    </b-tab-item>
                </b-tabs>
            </div>
        </template>
    </section>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import { BTable, BTabs, BTabItem } from 'buefy'

export interface PropInfo {
    name: string;
    description: string;
    type: string;
    values: string;
    default: string;
}

export interface SlotInfo {
    name: string;
    description: string;
    props?: string;
}

export interface EventInfo {
    name: string;
    description: string;
    parameters?: string;
}

export interface MethodInfo {
    name: string;
    description: string;
    parameters?: string;
    return?: string;
}

export interface ComponentInfo {
    title?: string;
    props?: PropInfo[];
    slots?: SlotInfo[];
    events?: EventInfo[];
    methods?: MethodInfo[];
}

export default defineComponent({
    components: {
        BTable,
        BTabs,
        BTabItem
    },
    props: {
        data: Array<ComponentInfo>
    },
    data() {
        return {
            propsColumns: [
                { label: 'Name', field: 'name', renderHtml: true },
                {
                    label: 'Description',
                    field: 'description',
                    renderHtml: true
                },
                { label: 'Type', field: 'type' },
                { label: 'Values', field: 'values', renderHtml: true },
                { label: 'Default', field: 'default', renderHtml: true }
            ],
            slotsColumns: [
                { label: 'Slot name', field: 'name', renderHtml: true },
                {
                    label: 'Description',
                    field: 'description',
                    renderHtml: true
                },
                {
                    label: 'Props (if scoped)',
                    field: 'props',
                    renderHtml: true
                }
            ],
            eventsColumns: [
                { label: 'Name', field: 'name', renderHtml: true },
                {
                    label: 'Description',
                    field: 'description',
                    renderHtml: true
                },
                { label: 'Parameters', field: 'parameters', renderHtml: true }
            ],
            methodsColumns: [
                { label: 'Name', field: 'name', renderHtml: true },
                {
                    label: 'Description',
                    field: 'description',
                    renderHtml: true
                },
                { label: 'Parameters', field: 'parameters', renderHtml: true },
                { label: 'Return', field: 'return', renderHtml: true }
            ]
        }
    }
})
</script>
