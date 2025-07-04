<template>
    <div
        class="autocomplete control"
        :class="{ 'is-expanded': expanded }"
        v-bind="rootAttrs"
    >
        <b-input
            v-model="newValue"
            ref="input"
            :type="type"
            :size="size"
            :loading="loading"
            :rounded="rounded"
            :icon="icon"
            :icon-right="newIconRight"
            :icon-right-clickable="newIconRightClickable"
            :icon-pack="iconPack"
            :maxlength="maxlength"
            :autocomplete="newAutocomplete"
            :use-html5-validation="false"
            :aria-autocomplete="ariaAutocomplete"
            v-bind="fallthroughAttrs"
            @update:model-value="onInput"
            @focus="focused"
            @blur="onBlur"
            @keydown="keydown"
            @keydown.up.prevent="keyArrows('up')"
            @keydown.down.prevent="keyArrows('down')"
            @icon-right-click="rightIconClick"
            @icon-click="(event: MouseEvent) => $emit('icon-click', event)"
        />

        <transition name="fade">
            <div
                class="dropdown-menu"
                :class="{ 'is-opened-top': isOpenedTop && !appendToBody }"
                :style="style"
                v-show="isActive && (!isEmpty || hasEmptySlot || hasHeaderSlot || hasFooterSlot)"
                ref="dropdown"
            >
                <div
                    class="dropdown-content"
                    v-show="isActive"
                    :style="contentStyle"
                >
                    <div
                        v-if="hasHeaderSlot"
                        class="dropdown-item dropdown-header"
                        role="button"
                        tabindex="0"
                        :class="{ 'is-hovered': headerHovered }"
                        @click="selectHeaderOrFoterByClick($event, 'header')"
                    >
                        <slot name="header" />
                    </div>
                    <template v-for="(element, groupindex) in computedData">
                        <div
                            v-if="element.group"
                            :key="groupindex + 'group'"
                            class="dropdown-item"
                        >
                            <slot
                                v-if="hasGroupSlot"
                                name="group"
                                :group="element.group"
                                :index="groupindex"
                            />
                            <span class="has-text-weight-bold" v-else>
                                {{ element.group }}
                            </span>
                        </div>
                        <a
                            v-for="(option, index) in element.items"
                            :key="groupindex + ':' + index"
                            class="dropdown-item"
                            role="button"
                            tabindex="0"
                            :class="{ 'is-hovered': option === hovered }"
                            @click.stop="setSelected(option, !keepOpen, $event)"
                        >
                            <slot
                                v-if="hasDefaultSlot"
                                :option="option"
                                :index="index"
                            />
                            <span v-else>
                                {{ getValue(option) }}
                            </span>
                        </a>
                    </template>
                    <div
                        v-if="isEmpty && hasEmptySlot"
                        class="dropdown-item is-disabled"
                    >
                        <slot name="empty" />
                    </div>
                    <div
                        v-if="hasFooterSlot"
                        class="dropdown-item dropdown-footer"
                        role="button"
                        tabindex="0"
                        :class="{ 'is-hovered': footerHovered }"
                        @click="selectHeaderOrFoterByClick($event, 'footer')"
                    >
                        <slot name="footer" />
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import type { PropType } from 'vue'

import {
    getValueByPath,
    removeElement,
    createAbsoluteElement,
    isCustomElement,
    toCssWidth
} from '../../utils/helpers'
import CompatFallthroughMixin from '../../utils/CompatFallthroughMixin'
import FormElementMixin from '../../utils/FormElementMixin'
import BInput from '../input/Input.vue'

type BInputComponent = InstanceType<typeof BInput>
// `$data` interface of `Taginput` component.
// we cannot directly import `Taginput` because it depends on `Autocomplete`.
interface TaginputData {
    _isTaginput: boolean
}

interface DataItem {
    group?: string
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    items: any[]
}

export default defineComponent({
    name: 'BAutocomplete',
    components: { BInput },
    mixins: [CompatFallthroughMixin, FormElementMixin],
    props: {
        modelValue: [Number, String, null] as PropType<number | string | null>,
        data: {
            type: Array,
            default: () => []
        },
        field: {
            type: String,
            default: 'value'
        },
        keepFirst: Boolean,
        clearOnSelect: Boolean,
        openOnFocus: Boolean,
        customFormatter: {
            // eslint-disable-next-line @typescript-eslint/no-explicit-any
            type: Function as PropType<(option: any) => string>
        },
        checkInfiniteScroll: Boolean,
        keepOpen: Boolean,
        selectOnClickOutside: Boolean,
        clearable: Boolean,
        maxHeight: [String, Number],
        dropdownPosition: {
            type: String,
            default: 'auto'
        },
        groupField: String,
        groupOptions: String,
        iconRight: String,
        iconRightClickable: Boolean,
        appendToBody: Boolean,
        type: {
            type: String,
            default: 'text'
        },
        confirmKeys: {
            type: Array,
            default: () => ['Tab', 'Enter']
        },
        selectableHeader: Boolean,
        selectableFooter: Boolean,
        // Native options to use in HTML5 validation
        autocomplete: String
    },
    emits: {
        /* eslint-disable @typescript-eslint/no-unused-vars, @typescript-eslint/no-explicit-any */
        active: (active: boolean) => true,
        blur: (event: Event) => true,
        focus: (event?: Event) => true,
        'icon-click': (event: MouseEvent) => true,
        'icon-right-click': (event: MouseEvent) => true,
        'infinite-scroll': () => true,
        select: (selected: any, event?: Event) => true,
        'select-footer': (event: Event) => true,
        'select-header': (event: Event) => true,
        typing: (value: number | string | null | undefined) => true,
        'update:modelValue': (value: number | string) => true
        /* eslint-enable @typescript-eslint/no-unused-vars, @typescript-eslint/no-explicit-any */
    },
    data() {
        return {
            // eslint-disable-next-line @typescript-eslint/no-explicit-any
            selected: null as any,
            // eslint-disable-next-line @typescript-eslint/no-explicit-any
            hovered: null as any,
            headerHovered: null as boolean | null,
            footerHovered: null as boolean | null,
            isActive: false,
            newValue: this.modelValue,
            newAutocomplete: this.autocomplete || 'off',
            ariaAutocomplete: this.keepFirst ? 'both' : 'list',
            isListInViewportVertically: true,
            hasFocus: false,
            style: {},
            _isAutocomplete: true,
            _elementRef: 'input',
            _bodyEl: undefined as Element | undefined, // Used to append to body
            timeOutID: undefined as ReturnType<typeof setTimeout> | undefined
        }
    },
    computed: {
        computedData(): DataItem[] {
            const { groupField, groupOptions } = this
            if (groupField) {
                if (groupOptions) {
                    const newData: DataItem[] = []
                    this.data.forEach((option) => {
                        const group = getValueByPath(option, groupField)
                        const items = getValueByPath(option, groupOptions)
                        newData.push({ group, items })
                    })
                    return newData
                } else {
                    // eslint-disable-next-line @typescript-eslint/no-explicit-any
                    const tmp: Record<string, any[]> = {}
                    this.data.forEach((option) => {
                        const group = getValueByPath(option, groupField)
                        if (!tmp[group]) tmp[group] = []
                        tmp[group].push(option)
                    })
                    const newData: DataItem[] = []
                    Object.keys(tmp).forEach((group) => {
                        newData.push({ group, items: tmp[group] })
                    })
                    return newData
                }
            }
            return [{ items: this.data }]
        },
        isEmpty() {
            if (!this.computedData) return true
            return !this.computedData.some((element) => element.items && element.items.length)
        },
        /*
         * White-listed items to not close when clicked.
         * Add input, dropdown and all children.
         */
        whiteList() {
            const whiteList = []
            whiteList.push((this.$refs.input as BInputComponent).$el.querySelector('input'))
            whiteList.push(this.$refs.dropdown)
            // Add all children from dropdown
            if (this.$refs.dropdown != null) {
                const children = (this.$refs.dropdown as Element).querySelectorAll('*')
                for (const child of children) {
                    whiteList.push(child)
                }
            }
            if ((this.$parent?.$data as TaginputData)._isTaginput) {
                // Add taginput container
                whiteList.push(this.$parent!.$el)
                // Add .tag and .delete
                const tagInputChildren = this.$parent!.$el.querySelectorAll('*')
                for (const tagInputChild of tagInputChildren) {
                    whiteList.push(tagInputChild)
                }
            }

            return whiteList
        },

        /*
         * Check if exists default slot
         */
        hasDefaultSlot() {
            return !!this.$slots.default
        },

        /*
         * Check if exists group slot
         */
        hasGroupSlot() {
            return !!this.$slots.group
        },

        /*
         * Check if exists "empty" slot
         */
        hasEmptySlot() {
            return !!this.$slots.empty
        },

        /*
         * Check if exists "header" slot
         */
        hasHeaderSlot() {
            return !!this.$slots.header
        },

        /*
         * Check if exists "footer" slot
         */
        hasFooterSlot() {
            return !!this.$slots.footer
        },

        /*
         * Apply dropdownPosition property
         */
        isOpenedTop() {
            return (
                this.dropdownPosition === 'top' ||
                    (this.dropdownPosition === 'auto' && !this.isListInViewportVertically)
            )
        },

        newIconRight() {
            if (this.clearable && this.newValue) {
                return 'close-circle'
            }
            return this.iconRight
        },

        newIconRightClickable() {
            if (this.clearable) {
                return true
            }
            return this.iconRightClickable
        },

        contentStyle() {
            return {
                maxHeight: toCssWidth(this.maxHeight) || undefined
            }
        }
    },
    watch: {
        /*
         * When dropdown is toggled, check the visibility to know when
         * to open upwards.
         */
        isActive(active) {
            if (this.dropdownPosition === 'auto') {
                if (active) {
                    this.calcDropdownInViewportVertical()
                } else {
                    // Timeout to wait for the animation to finish before recalculating
                    this.timeOutID = setTimeout(() => {
                        this.calcDropdownInViewportVertical()
                    }, 100)
                }
            }

            this.$nextTick(() => {
                this.$emit('active', active)
            })
        },

        /*
         * When checkInfiniteScroll property changes scroll event should be removed or added
         */
        checkInfiniteScroll(checkInfiniteScroll) {
            if ((this.$refs.dropdown && (this.$refs.dropdown as Element).querySelector('.dropdown-content')) === false) return

            const list = (this.$refs.dropdown as Element).querySelector('.dropdown-content')!

            if (checkInfiniteScroll === true) {
                list.addEventListener('scroll', this.checkIfReachedTheEndOfScroll)

                return
            }

            list.removeEventListener('scroll', this.checkIfReachedTheEndOfScroll)
        },

        /*
         * When updating input's value
         *   1. Emit changes
         *   2. If value isn't the same as selected, set null
         *   3. Close dropdown if value is clear or else open it
         */
        newValue(value) {
            this.$emit('update:modelValue', value)
            // Check if selected is invalid
            const currentValue = this.getValue(this.selected)
            if (currentValue && currentValue !== value) {
                this.setSelected(null, false)
            }
            // Close dropdown if input is clear or else open it
            if (this.hasFocus && (!this.openOnFocus || value)) {
                this.isActive = !!value
            }
        },

        /*
         * When v-model is changed:
         *   1. Update internal value.
         *   2. If it's invalid, validate again.
         */
        modelValue(value) {
            this.newValue = value
        },

        /*
         * Select first option if "keep-first
         */
        data() {
            // Keep first option always pre-selected
            if (this.keepFirst) {
                this.$nextTick(() => {
                    if (this.isActive) {
                        this.selectFirstOption(this.computedData)
                    } else {
                        this.setHovered(null)
                    }
                })
            } else {
                if (this.hovered) {
                    // reset hovered if list doesn't contain it
                    const hoveredValue = this.getValue(this.hovered)
                    const data = this.computedData.map((d) => d.items)
                        .reduce((a, b) => ([...a, ...b]), [])
                    if (!data.some((d) => this.getValue(d) === hoveredValue)) {
                        this.setHovered(null)
                    }
                }
            }
        }
    },
    methods: {
        /*
         * Set which option is currently hovered.
         */
        // eslint-disable-next-line @typescript-eslint/no-explicit-any
        setHovered(option: any) {
            if (option === undefined) return

            this.hovered = option
        },

        /*
         * Set which option is currently selected, update v-model,
         * update input value and close dropdown.
         */
        // eslint-disable-next-line @typescript-eslint/no-explicit-any
        setSelected(option: any, closeDropdown = true, event?: Event) {
            if (option === undefined) return
            this.selected = option
            this.$emit('select', this.selected, event)
            if (this.selected !== null) {
                if (this.clearOnSelect) {
                    const input = this.$refs.input as BInputComponent
                    input.newValue = ''
                    const innerInput = input.$refs.input as HTMLInputElement | HTMLTextAreaElement
                    innerInput.value = ''
                } else {
                    this.newValue = this.getValue(this.selected)
                }
                this.setHovered(null)
            }
            closeDropdown && this.$nextTick(() => {
                this.isActive = false
            })
            this.checkValidity()
        },

        /*
         * Select first option
         */
        selectFirstOption(computedData: DataItem[]) {
            this.$nextTick(() => {
                const nonEmptyElements = computedData.filter(
                    (element) => element.items && element.items.length
                )
                if (nonEmptyElements.length) {
                    const option = nonEmptyElements[0].items[0]
                    this.setHovered(option)
                } else {
                    this.setHovered(null)
                }
            })
        },

        keydown(event: KeyboardEvent) {
            const { key } = event // cannot destructure preventDefault (https://stackoverflow.com/a/49616808/2774496)
            // prevent emit submit event
            if (key === 'Enter') event.preventDefault()
            // Close dropdown on Tab & no hovered
            if (key === 'Escape' || key === 'Tab') {
                this.isActive = false
            }

            if (this.confirmKeys.indexOf(key) >= 0) {
                // If adding by comma, don't add the comma to the input
                if (key === ',') event.preventDefault()
                // Close dropdown on select by Tab
                const closeDropdown = !this.keepOpen || key === 'Tab'
                if (this.hovered === null) {
                    // header and footer uses headerHovered && footerHovered. If header or footer
                    // was selected then fire event otherwise just return so a value isn't selected
                    this.checkIfHeaderOrFooterSelected(event, null, closeDropdown)
                    return
                }
                this.setSelected(this.hovered, closeDropdown, event)
            }
        },

        selectHeaderOrFoterByClick(event: MouseEvent, origin: 'header' | 'footer') {
            this.checkIfHeaderOrFooterSelected(event, { origin })
        },

        /*
         * Check if header or footer was selected.
         */
        checkIfHeaderOrFooterSelected(
            event: Event,
            triggerClick: { origin: 'header' | 'footer' } | null,
            closeDropdown = true
        ) {
            if (this.selectableHeader && (this.headerHovered || (triggerClick && triggerClick.origin === 'header'))) {
                this.$emit('select-header', event)
                this.headerHovered = false
                if (triggerClick) this.setHovered(null)
                if (closeDropdown) this.isActive = false
            }
            if (this.selectableFooter && (this.footerHovered || (triggerClick && triggerClick.origin === 'footer'))) {
                this.$emit('select-footer', event)
                this.footerHovered = false
                if (triggerClick) this.setHovered(null)
                if (closeDropdown) this.isActive = false
            }
        },

        /*
         * Close dropdown if clicked outside.
         */
        clickedOutside(event: MouseEvent) {
            const target = isCustomElement(this) ? event.composedPath()[0] : event.target
            if (!this.hasFocus && this.whiteList.indexOf(target) < 0) {
                if (this.keepFirst && this.hovered && this.selectOnClickOutside) {
                    this.setSelected(this.hovered, true)
                } else {
                    this.isActive = false
                }
            }
        },

        /*
         * Return display text for the input.
         * If object, get value from path, or else just the value.
         */
        // eslint-disable-next-line @typescript-eslint/no-explicit-any
        getValue(option: any) {
            if (option === null) return

            if (typeof this.customFormatter !== 'undefined') {
                return this.customFormatter(option)
            }
            return typeof option === 'object' ? getValueByPath(option, this.field) : option
        },

        /*
         * Check if the scroll list inside the dropdown
         * reached it's end.
         */
        checkIfReachedTheEndOfScroll() {
            const list = (this.$refs.dropdown as Element).querySelector('.dropdown-content')!
            const footerHeight = this.hasFooterSlot
                ? list.querySelectorAll('div.dropdown-footer')[0].clientHeight
                : 0
            if (list.clientHeight !== list.scrollHeight &&
                (list.scrollTop +
                 list.parentElement!.clientHeight +
                 footerHeight) >= list.scrollHeight
            ) {
                this.$emit('infinite-scroll')
            }
        },

        /*
         * Calculate if the dropdown is vertically visible when activated,
         * otherwise it is openened upwards.
         */
        calcDropdownInViewportVertical() {
            this.$nextTick(() => {
                /*
                 * this.$refs.dropdown may be undefined
                 * when Autocomplete is conditional rendered
                 */
                if (this.$refs.dropdown == null) return

                const rect = (this.$refs.dropdown as Element).getBoundingClientRect()

                this.isListInViewportVertically = rect.top >= 0 &&
                    rect.bottom <= (window?.innerHeight || document?.documentElement?.clientHeight)
                if (this.appendToBody) {
                    this.updateAppendToBody()
                }
            })
        },

        /*
         * Arrows keys listener.
         * If dropdown is active, set hovered option, or else just open.
         */
        keyArrows(direction: 'down' | 'up') {
            const sum = direction === 'down' ? 1 : -1
            if (this.isActive) {
                const data = this.computedData.map(
                    (d) => d.items).reduce((a, b) => ([...a, ...b]), [])
                if (this.hasHeaderSlot && this.selectableHeader) {
                    data.unshift(undefined)
                }
                if (this.hasFooterSlot && this.selectableFooter) {
                    data.push(undefined)
                }

                let index
                if (this.headerHovered) {
                    index = 0 + sum
                } else if (this.footerHovered) {
                    index = (data.length - 1) + sum
                } else {
                    index = data.indexOf(this.hovered) + sum
                }

                index = index > data.length - 1 ? data.length - 1 : index
                index = index < 0 ? 0 : index

                this.footerHovered = false
                this.headerHovered = false
                this.setHovered(data[index] !== undefined ? data[index] : null)
                if (this.hasFooterSlot && this.selectableFooter && index === data.length - 1) {
                    this.footerHovered = true
                }
                if (this.hasHeaderSlot && this.selectableHeader && index === 0) {
                    this.headerHovered = true
                }

                const list = (this.$refs.dropdown as Element).querySelector('.dropdown-content')!
                let querySelectorText = 'a.dropdown-item:not(.is-disabled)'
                if (this.hasHeaderSlot && this.selectableHeader) {
                    querySelectorText += ',div.dropdown-header'
                }
                if (this.hasFooterSlot && this.selectableFooter) {
                    querySelectorText += ',div.dropdown-footer'
                }
                const element =
                    list.querySelectorAll(querySelectorText)[index] as HTMLElement | null

                if (!element) return

                const visMin = list.scrollTop
                const visMax = list.scrollTop + list.clientHeight - element.clientHeight

                if (element.offsetTop < visMin) {
                    list.scrollTop = element.offsetTop
                } else if (element.offsetTop >= visMax) {
                    list.scrollTop = element.offsetTop - list.clientHeight + element.clientHeight
                }
            } else {
                this.isActive = true
            }
        },

        /*
         * Focus listener.
         * If value is the same as selected, select all text.
         */
        focused(event?: Event) {
            if (this.getValue(this.selected) === this.newValue) {
                this.$el.querySelector('input').select()
            }
            if (this.openOnFocus) {
                this.isActive = true
                if (this.keepFirst) {
                    // If open on focus, update the hovered
                    this.selectFirstOption(this.computedData)
                }
            }
            this.hasFocus = true
            this.$emit('focus', event)
        },

        /*
         * Blur listener.
         */
        onBlur(event: Event) {
            this.hasFocus = false
            this.$emit('blur', event)
        },
        onInput() {
            const currentValue = this.getValue(this.selected)
            if (currentValue && currentValue === this.newValue) return
            this.$emit('typing', this.newValue)
            this.checkValidity()
        },
        rightIconClick(event: MouseEvent) {
            if (this.clearable) {
                this.newValue = ''
                this.setSelected(null, false)
                if (this.openOnFocus) {
                    (this.$refs.input as BInputComponent).$el.focus()
                }
            } else {
                this.$emit('icon-right-click', event)
            }
        },
        checkValidity() {
            if (this.useHtml5Validation) {
                this.$nextTick(() => {
                    this.checkHtml5Validity()
                })
            }
        },
        updateAppendToBody() {
            const dropdownMenu = this.$refs.dropdown as Element
            const trigger = (this.$parent!.$data as TaginputData)._isTaginput
                ? this.$parent!.$el
                : (this.$refs.input as BInputComponent).$el
            if (dropdownMenu && trigger) {
                // update wrapper dropdown
                const root = this.$data._bodyEl!
                root.classList.forEach((item) => root.classList.remove(item))
                root.classList.add('autocomplete')
                root.classList.add('control')
                if (this.expanded) {
                    root.classList.add('is-expanded')
                }
                const rect = trigger.getBoundingClientRect()
                let top = rect.top + window.scrollY
                const left = rect.left + window.scrollX
                if (!this.isOpenedTop) {
                    top += trigger.clientHeight
                } else {
                    top -= dropdownMenu.clientHeight
                }
                this.style = {
                    position: 'absolute',
                    top: `${top}px`,
                    left: `${left}px`,
                    width: `${trigger.clientWidth}px`,
                    maxWidth: `${trigger.clientWidth}px`,
                    zIndex: '99'
                }
            }
        }
    },
    created() {
        if (typeof window !== 'undefined') {
            document.addEventListener('click', this.clickedOutside)
            if (this.dropdownPosition === 'auto') { window.addEventListener('resize', this.calcDropdownInViewportVertical) }
        }
    },
    mounted() {
        if (this.checkInfiniteScroll &&
            this.$refs.dropdown && (this.$refs.dropdown as Element).querySelector('.dropdown-content')
        ) {
            const list = (this.$refs.dropdown as Element).querySelector('.dropdown-content')!
            list.addEventListener('scroll', this.checkIfReachedTheEndOfScroll)
        }
        if (this.appendToBody) {
            this.$data._bodyEl = createAbsoluteElement(this.$refs.dropdown as Element)
            this.updateAppendToBody()
        }
    },
    beforeUnmount() {
        if (typeof window !== 'undefined') {
            document.removeEventListener('click', this.clickedOutside)
            if (this.dropdownPosition === 'auto') { window.removeEventListener('resize', this.calcDropdownInViewportVertical) }
        }
        if (this.checkInfiniteScroll &&
            this.$refs.dropdown && (this.$refs.dropdown as Element).querySelector('.dropdown-content')
        ) {
            const list = (this.$refs.dropdown as Element).querySelector('.dropdown-content')!
            list.removeEventListener('scroll', this.checkIfReachedTheEndOfScroll)
        }
        if (this.appendToBody) {
            removeElement(this.$data._bodyEl!)
        }
        clearTimeout(this.timeOutID)
    }
})
</script>
