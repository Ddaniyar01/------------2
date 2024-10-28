<template>
    <div>
        <select v-if="items.length" v-model="selected" @change="emitSelection">
            <option v-for="item in items" :key="item.name" :value="item">
                {{ item.name }}
            </option>
        </select>
        <div v-if="selected && hasChildren">
            <RecursiveSelect
                v-for="child in currentChildren"
                :key="child.name"
                :items="child.children"
                :parent-selection="currentPath"
                @select="emitSelectionFromChild"
            />
        </div>
    </div>
</template>

<script>
export default {
    name: 'RecursiveSelect',
    props: {
        items: {
            type: Array,
            required: true
        },
        parentSelection: {
            type: Array,
            default: () => []
        }
    },
    data() {
        return {
            selected: null,
            selectedItem: null
        };
    },
    computed: {
        hasChildren() {
            return this.currentChildren && this.currentChildren.length > 0;
        },
        currentChildren() {
            return this.items.filter(item => this.selected.name === item.name);
        },
        currentPath() {
            return this.selected ? [...this.parentSelection, this.selected.name] : this.parentSelection;
        }
    },
    methods: {
        emitSelection() {
            this.$emit('select', this.currentPath); // Отправляем событие с выбранным элементом
        },
        emitSelectionFromChild(selection) {
            this.$emit('select', selection); // Emit to the parent for any child selection
        },
    }
};
</script>

<style>
select {
    margin-right: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
    padding: 10px;
    font-size: 16px;
    width: 98%;
}
</style>
