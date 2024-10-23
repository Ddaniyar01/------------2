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
      <a href="#products" class="btn" @click="openAddProductModal">Каталог товаров</a>
    </section>

    <section id="products" class="products">
      <h2>Наши товары</h2>

      <button @click="openAddProductModal" class="add-btn">Добавить товар</button>

      <table>
        <thead>
          <tr>
            <th>Деталь</th>
            <th>Цена</th>
            <th>Количество</th>
            <th>Стоимость</th>
            <th>Действия</th>
          </tr>
        </thead>
        <tbody>
          <template v-for="item in items" :key="item.name">
            <tr class="parent-item">
              <td>{{ item.name }}</td>
              <td>{{ item.price }}</td>
              <td>{{ item.quantity }}</td>
              <td>{{ calculateTotal(item) }}</td>
              <td>
                <button @click="editItem(item, item)">Редактировать</button>
              </td>
            </tr>
            <template v-for="child in item.children" :key="child.name">
              <tr class="child-item">
                <td style="padding-left: 20px;">{{ child.name }}</td>
                <td>{{ child.price }}</td>
                <td>{{ child.quantity }}</td>
                <td>{{ calculateTotal(child) }}</td>
                <td>
                  <button @click="editItem(item, child)">Редактировать</button>
                </td>
              </tr>
              <template v-for="grandChild in child.children" :key="grandChild.name">
                <tr class="grandchild-item">
                  <td style="padding-left: 40px;">{{ grandChild.name }}</td>
                  <td>{{ grandChild.price }}</td>
                  <td>{{ grandChild.quantity }}</td>
                  <td>{{ calculateTotal(grandChild) }}</td>
                  <td>
                    <button @click="editItem(item, grandChild)">Редактировать</button>
                  </td>
                </tr>
              </template>
            </template>
          </template>
        </tbody>
      </table>

      <!-- Модальное окно для добавления товара -->
      <div v-if="isAddProductModalOpen" class="modal-overlay">
        <div class="modal">
          <h2>Добавить товар</h2>
          <div class="add-product-form">
            <input v-model="newItemName" placeholder="Название товара" />
            <input type="number" v-model="newItemPrice" placeholder="Цена" />
            <input type="number" v-model="newItemQuantity" placeholder="Количество" />
            <select v-model="parentItem">
              <option selected disabled value="">Выберите родительский товар</option>
              <option v-for="item in items" :key="item.name" :value="item">{{ item.name }}</option>
            </select>
            <select v-if="parentItem" v-model="childItem">
              <option selected disabled value="">Выберите подродительский товар</option>
              <option v-for="child in parentItem.children" :key="child.name" :value="child">{{ child.name }}</option>
            </select>
            <select v-if="childItem" v-model="grandChildItem">
              <option selected disabled value="">Выберите подподродительский товар</option>
              <option v-for="grandChild in childItem.children" :key="grandChild.name" :value="grandChild">{{ grandChild.name }}</option>
            </select>
            <button @click="addItem" class="add-btn">Добавить товар</button>
            <button @click="closeAddProductModal">Отмена</button>
          </div>
        </div>
      </div>

      <!-- Редактирование товара -->
      <div v-if="editingItem" class="edit-product-form">
        <h2>Редактировать товар</h2>
        <input v-model="editingItem.name" placeholder="Название товара" />
        <input type="number" v-model="editingItem.price" placeholder="Цена" />
        <input type="number" v-model="editingItem.quantity" @input="validateQuantity" placeholder="Количество" />
        
        <select v-model="newParentItem">
          <option selected disabled value="">Выберите нового родительского товар</option>
          <option v-for="item in items" :key="item.name" :value="item">{{ item.name }}</option>
        </select>

        <select v-if="newParentItem" v-model="newChildItem">
          <option selected disabled value="">Выберите нового подродительского товар</option>
          <option v-for="child in newParentItem.children" :key="child.name" :value="child">{{ child.name }}</option>
        </select>

        <select v-if="newChildItem" v-model="newGrandChildItem">
          <option selected disabled value="">Выберите нового подподродительского товар</option>
          <option v-for="grandChild in newChildItem.children" :key="grandChild.name" :value="grandChild">{{ grandChild.name }}</option>
        </select>

        <button @click="saveEdit">Сохранить изменения</button>
        <button @click="removeItem(editingItem)">Удалить</button>
        <button @click="closeEditModal">Отмена</button>
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
export default {
  data() {
    return {
      items: this.loadItems(),
      newItemName: '',
      newItemPrice: null,
      newItemQuantity: null,
      parentItem: null,
      childItem: null,
      editingItem: null,
      isAddProductModalOpen: false,
      newParentItem: null,
      newChildItem: null,
      newGrandChildItem: null
    };
  },
  mounted() {
    this.sumItems();
  },
  methods: {
    loadItems() {
      const savedItems = localStorage.getItem('items');
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
              quantity: 5,
              children: [

              ]
            },
            {
              name: "Полики",
              price: 2000,
              quantity: 3,
              children: []
              
            }
          ]
        }
      ];
    },
    saveItems() {
      localStorage.setItem('items', JSON.stringify(this.items));
    },
    addItem() {
      if (!this.newItemName || this.newItemPrice === null || this.newItemQuantity === null) {
        alert('Пожалуйста, заполните все поля!');
        return;
      }

      const newItem = {
        name: this.newItemName,
        price: this.newItemPrice,
        quantity: this.newItemQuantity,
        children: []
      };

      if (this.parentItem) {
        if (this.childItem) {
          if (this.newGrandChildItem) {
            this.newGrandChildItem.children.push(newItem);
          } else {
            this.childItem.children.push(newItem);
          }
        } else {
          this.parentItem.children.push(newItem);
        }
      } else {
        this.items.push(newItem);
      }

      this.sumItems();
      this.saveItems();
      this.closeAddProductModal();
    },
    editItem(items, item) {
      this.editingItem = item;
      this.newParentItem = null;
      this.newChildItem = null;
      this.newGrandChildItem = null;
    },
    saveEdit() {
  const newItem = {
    name: this.editingItem.name,
    price: this.editingItem.price,
    quantity: this.editingItem.quantity,
    children: this.editingItem.children || [] // Сохраняем вложенных детей, если есть
  };
  
  // Удаляем элемент из текущего места
  this.removeItem(this.editingItem);

  // Проверяем, куда переместить элемент
  if (this.newGrandChildItem) {
    this.newGrandChildItem.children.push(newItem); // Добавляем как внука
  } else if (this.newChildItem) {
    this.newChildItem.children.push(newItem); // Добавляем как ребенка
  } else if (this.newParentItem) {
    this.newParentItem.children.push(newItem); // Добавляем как дочерний элемент родителя
  } else {
    this.items.push(newItem); // Если нет родителя, добавляем как корневой элемент
  }

  this.sumItems(); // Пересчитываем суммы
  this.saveItems(); // Сохраняем данные
  this.closeEditModal(); // Закрываем модальное окно
},
    removeItem(item) {
      const recursiveRemove = (items, target) => {
    return items.filter(i => {
      if (i === target) return false;
      if (i.children) {
        i.children = recursiveRemove(i.children, target);
      }
      return true;
    });
  };


  this.items = recursiveRemove(this.items, item);

  // После удаления обновляем итоги и сохраняем изменения
  this.sumItems();
  this.saveItems();
},

findParent(item) {
      return this.items.find(i => i.children && i.children.includes(item));
    },
    calculateTotal(item) {
      // Рекурсивно суммируем стоимость текущего элемента и всех его потомков
      let total = item.price * item.quantity;
      if (item.children) {
        total += item.children.reduce((sum, child) => sum + this.calculateTotal(child), 0);
      }
      return total;
    },
    sumItems() {
      this.items.forEach(item => {
        let parent_price = 0;
        if (item.children && item.children.length > 0) {
          item.children.forEach(child => {
            let child_price = 0;
            if (child.children && child.children.length > 0) {
              child.children.forEach(childish => {
                child_price += childish.quantity * childish.price;
              });
              child.price = child_price;
            }
            parent_price += child.quantity * child.price;
          });
          item.price = parent_price;
        };
      })
    },
    validateQuantity() {
      if (this.editingItem.quantity < 0) {
        alert('Количество не может быть отрицательным!');
        this.editingItem.quantity = 0;
      }
    },
    openAddProductModal() {
      this.isAddProductModalOpen = true;
      this.newItemName = '';
      this.newItemPrice = null;
      this.newItemQuantity = null;
      this.parentItem = null;
      this.childItem = null;
    },
    closeAddProductModal() {
      this.isAddProductModalOpen = false;
    },
    closeEditModal() {
      this.editingItem = null;
    }
  }
};
</script>


<style scoped>
body {
  font-family: 'Arial', sans-serif;
  background-color: #f4f4f4;
  margin: 0;
  padding: 0;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: #333;
  color: #fff;
  padding: 20px;
}

nav ul {
  list-style: none;
  display: flex;
}

nav ul li {
  margin: 0 15px;
}

nav ul li a {
  color: #fff;
  text-decoration: none;
}

.hero {
  text-align: center;
  padding: 50px;
  background-color: #eaeaea;
  border-bottom: 5px solid #333;
}

.products {
  padding: 20px;
}

.add-product-form {
  margin-bottom: 20px;
}

.add-product-form input,
.add-product-form select {
  margin-right: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  padding: 10px;
  font-size: 16px;
}

.add-product-form .add-btn {
  background-color: #333;
  color: #fff;
  border: none;
  border-radius: 5px;
  padding: 10px 20px;
  cursor: pointer;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

th,
td {
  border: 1px solid #ccc;
  padding: 10px;
  text-align: left;
}

th {
  background-color: #333;
  color: #fff;
}

td {
  transition: background-color 0.3s;
}

td:hover {
  background-color: #f0f0f0;
}

.parent-item {
  background-color: #e2e2e2;
}

.child-item {
  background-color: #fff;
  border-left: 5px solid #6c757d; /* Темно-серая линия слева */
}

.grandchild-item {
  background-color: #f9f9f9;
  border-left: 5px solid #007bff; /* Синяя линия слева */
}

.edit-product-form {
  margin-top: 20px;
  padding: 20px;
  background-color: #eaeaea;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.contact {
  padding: 20px;
  background-color: #fff;
  border-top: 5px solid #333;
}

footer {
  text-align: center;
  padding: 10px;
  background-color: #333;
  color: #fff;
}
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
}

.modal {
  background: white;
  padding: 20px;
  border-radius: 5px;
  max-width: 500px;
  width: 100%;
}

.add-product-form {
  display: flex;
  flex-direction: column;
}

.add-btn {
  margin-top: 10px;
}
</style>