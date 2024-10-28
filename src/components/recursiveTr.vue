<template>
    <tr>
        <td :style="{ paddingLeft: level * 20 + 'px' }">{{ data.name }}</td>
        <td>{{ data.price }}</td>
        <td>{{ data.quantity }}</td>
      <td>{{ Math.floor(data.price * data.quantity) }}</td> <!-- Обновлено для целых чисел -->
        <td>
            <button @click="$emit('edit', data)">Редактировать</button>
        </td>
    </tr>
    <template v-for="child in data.children" :key="child.name">
        <RecursiveTr :data="child" :level="level + 1" @edit="$emit('edit', $event)" />
    </template>
</template>

<script>
export default {
    name: 'RecursiveTr',
    props: {
        data: {
            type: Object,
            required: true
        },
        level: {
            type: Number,
            required: false,
            default: 0
        },
        items: {
            type: Array,
            required: true
        },
    },
    mounted() {
        this.calculateTotal(this.data);
        this.$watch('items', () => {
            this.calculateTotal(this.data);
        });
    },
    methods: {
        getRowClass(level) {
            return level === 0 ? 'parent-item' : level === 1 ? 'child-item' : 'grandchild-item';
        },
        
        editItem() {
            this.$emit('edit', this.data); // Эмитируем событие редактирования
        },
        removeItem() {
            this.$emit('remove', this.data); // Эмитируем событие удаления
        },
        calculateTotal(item) {
            if (item.children && item.children.length > 0) {
                item.price = 0;
                for (const child of item.children) {
                    item.price += child.price * child.quantity;
                    this.calculateTotal(child);
                }
            }
        }
    }
};

</script>

<style scoped>

body {
  font-family: 'Arial', sans-serif;
  background-color: #f0f0f5;
  margin: 0;
  padding: 0;
  color: #333;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: #2c3e50;
  color: #ecf0f1;
  padding: 15px 30px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

nav ul {
  list-style: none;
  display: flex;
  padding: 0;
}

nav ul li {
  margin: 0 15px;
}

nav ul li a {
  color: #ecf0f1;
  text-decoration: none;
  font-weight: bold;
}

nav ul li a:hover {
  text-decoration: underline;
}

.hero {
  text-align: center;
  padding: 60px;
  background: linear-gradient(to right, #e3e4e8, #ffffff);
  border-bottom: 5px solid #2c3e50;
}

.products {
  padding: 30px;
  max-width: 1200px;
  margin: 0 auto;
}

.add-product-form {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 20px;
}

.add-product-form input,
.add-product-form select {
  border: 1px solid #bdc3c7;
  border-radius: 5px;
  padding: 12px;
  font-size: 16px;
  flex: 1;
  min-width: 200px;
}

.add-product-form .add-btn {
  background-color: #2c3e50;
  color: #fff;
  border: none;
  border-radius: 5px;
  padding: 12px 24px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.add-product-form .add-btn:hover {
  background-color: #34495e;
}

/* Стиль таблицы с видимыми границами */
table {
  width: 100%;
  border-collapse: collapse; /* Убирает промежутки между границами */
  margin-top: 20px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  background-color: #fff;
}

th, td {
  border: 1px solid #ccc; /* Устанавливает границу для ячеек */
  padding: 12px;
  text-align: left;
}

th {
  background-color: #2c3e50;
  color: #fff;
  font-weight: bold;
}

tr:nth-child(even) td {
  background-color: #f9f9f9;
}

tr:hover td {
  background-color: #e0e0e0;
}



/* Стили для вложенных элементов */
.parent-item {
  background-color: #e6e6e6;
}

.child-item {
  background-color: #fafafa;
  border-left: 5px solid #6c757d;
}

.grandchild-item {
  background-color: #f1f1f1;
  border-left: 5px solid #3498db;
}

/* Форма редактирования */
.edit-product-form {
  margin-top: 20px;
  padding: 20px;
  background-color: #f8f8f8;
  border: 1px solid #ddd;
  border-radius: 5px;
}

/* Стили модального окна */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal {
  background: #fff;
  padding: 30px;
  border-radius: 8px;
  max-width: 500px;
  width: 100%;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.add-product-form {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.add-btn {
  margin-top: 10px;
}

/* Контакты и футер */
.contact {
  padding: 20px;
  background-color: #fff;
  border-top: 5px solid #2c3e50;
}

footer {
  text-align: center;
  padding: 15px;
  background-color: #2c3e50;
  color: #ecf0f1;
}
</style>
