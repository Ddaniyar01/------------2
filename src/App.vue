<template>
  <div>
    <header>
      <div class="logo">
        <h1>Магазин Автозапчастей</h1>
      </div>
      <nav>
        <ul>
          <li><a href="#home">Главная</a></li>
          <li><a href="#products">Товары</a></li>
          <li><a href="#contact">Контакты</a></li>
        </ul>
      </nav>
    </header>

    <section id="home" class="hero">
      <h2>Лучшие автозапчасти по доступным ценам!</h2>
      <p>Мы предлагаем качественные автозапчасти для всех марок автомобилей.</p>
      <a href="#products" class="btn">Каталог товаров</a>
    </section>

    <section id="products" class="products">
      <h2>Наши товары</h2>

      <div class="add-product-form">
        <input v-model="newItemName" placeholder="Название товара" />
        <input type="number" v-model="newItemPrice" placeholder="Цена" />
        <input type="number" v-model="newItemQuantity" placeholder="Количество" />
        <select v-model="parentItem">
          <option value="">Выберите родительский товар</option>
          <option v-for="item in itemes" :key="item.name" :value="item">{{ item.name }}</option>
        </select>
        <button @click="addItem" class="add-btn">Добавить товар</button>
      </div>

      <table>
        <thead>
          <tr>
            <th>Деталь</th>
            <th>Цена</th>
            <th>Количество</th>
            <th>Стоимость</th>
            <th>Действия</th>
            <th>Редактировать</th>
          </tr>
        </thead>
        <tbody>
          <template v-for="item in itemes" :key="item.name">
            <tr>
              <td :style="getIndentStyle(0)">{{ item.name }}</td>
              <td>{{ item.price }}</td>
              <td>{{ item.quantity }}</td>
              <td>{{ item.price * item.quantity }}</td>
              <td>
                <button @click="reduceQuantity(item)">Удалить</button>
              </td>
              <td>
                <button @click="editItem(item)">Редактировать</button>
              </td>
            </tr>

            <!-- Вложенные элементы -->
            <template v-for="child in item.children" :key="child.name">
              <tr>
                <td :style="getIndentStyle(1)">{{ child.name }}</td>
                <td>{{ child.price }}</td>
                <td>{{ child.quantity }}</td>
                <td>{{ child.price * child.quantity }}</td>
                <td>
                  <button @click="reduceQuantity(child)">Удалить</button>
                </td>
                <td>
                  <button @click="editItem(child)">Редактировать</button>
                </td>
              </tr>

              <!-- Дальнейшая вложенность -->
              <template v-for="childer in child.children" :key="childer.name">
                <tr>
                  <td :style="getIndentStyle(2)">{{ childer.name }}</td>
                  <td>{{ childer.price }}</td>
                  <td>{{ childer.quantity }}</td>
                  <td>{{ childer.price * childer.quantity }}</td>
                  <td>
                    <button @click="reduceQuantity(childer)">Удалить</button>
                  </td>
                  <td>
                    <button @click="editItem(childer)">Редактировать</button>
                  </td>
                </tr>
              </template>
            </template>
          </template>
        </tbody>
      </table>

      <div>
        <button @click="exportToExcel" class="export-btn">Экспортировать в Excel</button>
      </div>

      <!-- Форма редактирования -->
      <div v-if="editingItem" class="edit-product-form">
        <h3>Редактировать товар: {{ editingItem.name }}</h3>
        <input v-model="editingItem.name" placeholder="Название товара" />
        <input type="number" v-model="editingItem.price" placeholder="Цена" />
        <input type="number" v-model="editingItem.quantity" placeholder="Количество" />
        <button @click="saveItem" class="save-btn">Сохранить</button>
        <button @click="removeEditingItem" class="remove-btn">Удалить</button>
        <button @click="cancelEdit" class="cancel-btn">Отменить</button>
      </div>
    </section>

    <section id="contact" class="contact">
      <h2>Контакты</h2>
      <p>Телефон: +7 (777) 777-77-77</p>
      <p>Email: info@autozapchasti.kz</p>
      <p>Адрес: г. Астана, пр. Улы Дала, д. 10</p>
    </section>

    <footer>
      <p>&copy; 2024 Магазин Автозапчастей. Все права защищены.</p>
    </footer>
  </div>
</template>

<script>
import * as XLSX from 'xlsx';

export default {
  data() {
    return {
      itemes: this.loadItems(),
      newItemName: '',
      newItemPrice: null,
      newItemQuantity: null,
      parentItem: null,
      editingItem: null,
    };
  },
  mounted() {
    this.sumItems();
  },
  methods: {
    loadItems() {
      const savedItems = localStorage.getItem('itemes');
      return savedItems ? JSON.parse(savedItems) : this.defaultItems();
    },

    defaultItems() {
      return [
        {
          name: "Кузов",
          price: 0,
          quantity: 2,
          children: [
            {
              name: "Двери",
              price: 10000,
              quantity: 3,
              children: [
                {
                  name: "Замок",
                  price: 5000,
                  quantity: 4
                },
                {
                  name: "Ручки",
                  price: 6000,
                  quantity: 6
                }
              ]
            }
          ]
        },
        {
          name: "Двигатель",
          price: 0,
          quantity: 1,
          children: [
            {
              name: "Поршни",
              price: 10000,
              quantity: 5,
              children: [
                {
                  name: "Кольца",
                  price: 2000,
                  quantity: 3
                }
              ]
            }
          ]
        },
        {
          name: "Салон",
          price: 0,
          quantity: 1,
          children: [
            {
              name: "Чехлы",
              price: 10000,
              quantity: 5
            },
            {
              name: "Полики",
              price: 2000,
              quantity: 3
            }
          ]
        }
      ];
    },

    saveItems() {
      localStorage.setItem('itemes', JSON.stringify(this.itemes));
    },

    sumItems() {
      this.itemes.forEach(item => {
        item.price = item.children.reduce((acc, child) => {
          let childPrice = child.price * child.quantity;
          if (child.children) {
            childPrice += child.children.reduce((chAcc, childer) => chAcc + (childer.price * childer.quantity), 0);
          }
          return acc + childPrice;
        }, 0);
      });
      this.saveItems();
    },

    addQuantity(item) {
      item.quantity += 1;
      this.sumItems();
    },

    reduceQuantity(item) {
      if (item.quantity > 1) {
        item.quantity -= 1;
      } else {
        alert('Количество не может быть меньше 1');
      }
      this.sumItems();
    },

    addItem() {
      const newItem = {
        name: this.newItemName,
        price: Number(this.newItemPrice),
        quantity: Number(this.newItemQuantity),
        children: []
      };

      if (this.parentItem) {
        this.parentItem.children.push(newItem);
      } else {
        this.itemes.push(newItem);
      }
      this.resetNewItemFields();
      this.sumItems();
      this.updateParentItemOptions();
    },

    resetNewItemFields() {
      this.newItemName = '';
      this.newItemPrice = null;
      this.newItemQuantity = null;
      this.parentItem = null;
    },

    updateParentItemOptions() {
      // Обновление родительского элемента для корректного отображения
      this.parentItem = null;
    },

    editItem(item) {
      this.editingItem = item;
    },

    saveItem() {
      this.saveItems(); // Сохранение после редактирования
      this.editingItem = null;
      this.sumItems(); // Пересчет общей стоимости
    },

    removeEditingItem() {
      if (confirm('Вы уверены, что хотите удалить этот товар?')) {
        const parent = this.findParent(this.itemes, this.editingItem);
        if (parent) {
          const index = parent.children.indexOf(this.editingItem);
          parent.children.splice(index, 1);
        } else {
          const index = this.itemes.indexOf(this.editingItem);
          this.itemes.splice(index, 1);
        }
        this.editingItem = null;
        this.sumItems(); // Пересчет общей стоимости
      }
    },

    findParent(items, child) {
      for (const item of items) {
        if (item.children.includes(child)) {
          return item;
        }
        const found = this.findParent(item.children, child);
        if (found) return found;
      }
      return null;
    },

    cancelEdit() {
      this.editingItem = null;
    },

    exportToExcel() {
      const ws = XLSX.utils.json_to_sheet(this.itemes);
      const wb = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(wb, ws, 'Товары');
      XLSX.writeFile(wb, 'товары.xlsx');
    },

    getIndentStyle(level) {
      return {
        paddingLeft: `${level * 20}px`,
      };
    }
  }
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Arial', sans-serif;
}

body {
  background-color: #f9f9f9; /* Светлый фон для удобства восприятия */
  color: #333;
}

header {
  background-color: #3e3e3e; /* Тёмно-серый цвет */
  color: #ffffff; /* Белый текст для контраста */
  padding: 20px 0;
  text-align: center;
}

header .logo h1 {
  font-size: 2rem;
  margin: 0;
}

header nav ul {
  list-style: none;
  padding: 0;
  display: flex;
  justify-content: center;
  margin-top: 10px;
}

header nav ul li {
  margin: 0 20px;
}

header nav ul li a {
  color: #ffffff; /* Белый цвет для ссылок */
  text-decoration: none;
  font-size: 1.1rem;
  transition: color 0.3s;
}

header nav ul li a:hover {
  color: #f0f0f0; /* Чуть более светлый оттенок при наведении */
}

.hero {
  background-color: #424242; /* Тёмно-серый фон для раздела */
  padding: 50px;
  text-align: center;
  color: #ffffff; /* Белый текст для контраста */
}

.hero h2 {
  font-size: 2.5rem;
  margin-bottom: 20px;
}

.hero p {
  font-size: 1.2rem;
  margin-bottom: 20px;
}

.hero .btn {
  background-color: #ff5722; /* Яркий оранжевый цвет */
  color: white;
  padding: 10px 20px;
  border: none;
  cursor: pointer;
  font-size: 1.1rem;
  border-radius: 5px;
  transition: background-color 0.3s;
}

.hero .btn:hover {
  background-color: #e64a19; /* Более тёмный оттенок при наведении */
}

.products {
  padding: 50px;
  background-color: #ffffff; /* Белый фон для продуктов */
  margin-top: 20px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

.products h2 {
  text-align: center;
  font-size: 2rem;
  margin-bottom: 40px;
  color: #424242; /* Тёмно-серый для заголовка */
}

.add-product-form {
  display: flex;
  justify-content: space-around;
  margin-bottom: 30px;
  gap: 20px;
}

.add-product-form input,
.add-product-form select {
  padding: 10px;
  font-size: 1rem;
  border-radius: 5px;
  border: 1px solid #ccc; /* Светло-серый цвет рамки */
  color: #333;
}

.add-product-form .add-btn {
  background-color: #2196f3; /* Яркий синий цвет */
  color: white;
  padding: 10px 20px;
  border: none;
  cursor: pointer;
  border-radius: 5px;
  transition: background-color 0.3s;
}

.add-product-form .add-btn:hover {
  background-color: #1976d2; /* Более тёмный синий при наведении */
}

.table-container {
  max-width: 100%;
  overflow-x: auto;
  background-color: #ffffff; /* Белый фон для контейнера */
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

table {
  width: 100%;
  border-collapse: collapse;
}

table thead th {
  background-color: #e0e0e0; /* Светло-серый для заголовков */
  color: #424242;
  padding: 12px;
  text-align: left;
  font-size: 1.1rem;
}

table tbody tr {
  background-color: #fafafa; /* Очень светлый серый для строк */
}

table tbody tr:nth-child(even) {
  background-color: #f5f5f5; /* Легкий оттенок для четных строк */
}

table td {
  padding: 12px;
  border-bottom: 1px solid #ddd; /* Светло-серый цвет границы */
}

table td button {
  background-color: #ff5722; /* Яркий оранжевый цвет */
  color: white;
  padding: 5px 10px;
  border: none;
  border-radius: 3px;
  cursor: pointer;
  transition: background-color 0.3s;
}

table td button:hover {
  background-color: #e64a19; /* Более тёмный оттенок при наведении */
}

.edit-product-form {
  background-color: #f5f5f5; /* Светло-серый фон для формы редактирования */
  padding: 20px;
  margin-top: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

.edit-product-form input {
  padding: 10px;
  margin-bottom: 15px;
  width: 100%;
  border-radius: 5px;
  border: 1px solid #ccc; /* Светло-серый цвет рамки */
}

.edit-product-form button {
  background-color: #2196f3; /* Яркий синий цвет */
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  margin-right: 10px;
  transition: background-color 0.3s;
}

.edit-product-form button:hover {
  background-color: #1976d2; /* Более тёмный синий при наведении */
}

.contact {
  padding: 50px;
  background-color: #ffffff; /* Белый фон для контактов */
  margin-top: 20px;
}

.contact h2 {
  text-align: center;
  font-size: 2rem;
  margin-bottom: 40px;
  color: #424242; /* Тёмно-серый для заголовка */
}

.contact p {
  font-size: 1.2rem;
  text-align: center;
  margin-bottom: 15px;
}

footer {
  background-color: #3e3e3e; /* Тёмно-серый цвет */
  color: #ffffff; /* Белый текст для контраста */
  text-align: center;
  padding: 15px 0;
  margin-top: 50px;
}

footer p {
  margin: 0;
  font-size: 1rem;
}

</style>
