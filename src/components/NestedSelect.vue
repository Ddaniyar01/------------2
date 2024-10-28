<template>
  <div>
    <select @change="onSelect($event.target.value)">
      <option disabled selected value="">Выберите...</option>
      <option v-for="(item, index) in items" :key="index" :value="item.name">
        {{ item.name }}
      </option>
    </select>
    <div v-if="selectedItem && selectedItem.children && selectedItem.children.length > 0">
      <nested-select :items="selectedItem.children" @select="onSelect" />
    </div>
  </div>
</template>

<script>
export default {
  name: 'NestedSelect',
  props: {
    items: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      selectedItem: null,
    };
  },
  methods: {
    onSelect(value) {
      this.selectedItem = this.items.find(item => item.name === value);
      this.$emit('select', value); // Эмитируем выбранное значение
    },
  },
};
</script>
