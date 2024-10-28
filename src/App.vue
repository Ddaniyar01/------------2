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
            <p>
                Мы предлагаем качественные автозапчасти для всех марок
                автомобилей.
            </p>
            <a href="#products" class="btn" @click="openAddProductModal">
                Каталог товаров
            </a>
        </section>

        <section id="products" class="products">
            <h2>Наши товары</h2>

            <button @click="openAddProductModal" class="add-btn">
                Добавить товар
            </button>

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
                        <RecursiveTr :data="item" :level="0" :items="items" @edit="editItem" />
                    </template>
                </tbody>
            </table>

            <!-- Модальное окно для добавления товара -->
            <div v-if="isAddProductModalOpen" class="modal-overlay">
                <div class="modal">
                    <h2>Добавить товар</h2>
                    <div class="add-product-form">
                        <input v-model="newItem.name" placeholder="Название товара" />
                        <input type="number" v-model="newItem.price" placeholder="Цена" />
                        <input type="number" v-model="newItem.quantity" placeholder="Количество" />
                        <RecursiveSelect :items="items" @select="handleSelection" />
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
                    <option v-for="item in items" :key="item.name" :value="item">
                        {{ item.name }}
                    </option>
                </select>

                <select v-if="newParentItem" v-model="newChildItem">
                    <option selected disabled value="">Выберите нового подродительского товар</option>
                    <option v-for="child in newParentItem.children" :key="child.name" :value="child">
                        {{ child.name }}
                    </option>
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
import RecursiveSelect from './components/recursiveSelect.vue'
import RecursiveTr from './components/recursiveTr.vue'
export default {
    components: {
        RecursiveSelect,
        RecursiveTr
    },
    data() {
        return {
            items: this.loadItems(),
            parentItem: null,
            childItem: null,
            editingItem: null,
            isAddProductModalOpen: false,
            newItem: {
                name: '',
                price: null,
                quantity: null,
                children: []
            },
            selectedItem: null,
            newParentItem: null,
            newChildItem: null
        };
    },
    mounted() {},
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
                                    quantity: 4,
                                    children: []
                                },
                                {
                                    name: "Ручки",
                                    price: 6000,
                                    quantity: 6,
                                    children: []
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
                                    quantity: 3,
                                    children: []
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
                            children: []
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
        findItemByName(array, name) {
            for (const item of array) {
                if (item.name === name) {
                    return item;
                }
                if (item.children) {
                    const foundInChildren = this.findItemByName(item.children, name);
                    if (foundInChildren) {
                        return foundInChildren;
                    }
                }
            }
            return null;
        },
        addItem() {
            if (!this.newItem.name || this.newItem.price === null || this.newItem.quantity === null) {
                alert('Пожалуйста, заполните все поля!');
                return;
            }

            let selectedParent = null;
            if (this.selectedItem) {
                const result = this.findItemByName(this.items, this.selectedItem[this.selectedItem.length - 1]);
                result.children.push({ ...this.newItem });
            } else {
                this.items.push({ ...this.newItem });
            }

            this.saveItems();
            this.closeAddProductModal();
        },
        editItem(item) {
            this.editingItem = item;
            this.newParentItem = null; // Сброс нового родителя
            this.newChildItem = null;  // Сброс нового подродителя
        },
        saveEdit() {
            // Обновляем свойства существующего элемента
            this.editingItem.name = this.editingItem.name;
            this.editingItem.price = this.editingItem.price;
            this.editingItem.quantity = this.editingItem.quantity;

            // Если новый родитель или новый подродитель выбран
            if (this.newParentItem || this.newChildItem) {
                this.removeItem(this.editingItem); // Удаляем элемент из старого места

                if (this.newChildItem) {
                    this.newChildItem.children.push(this.editingItem); // Добавляем к новому подродителю
                } else if (this.newParentItem) {
                    this.newParentItem.children.push(this.editingItem); // Добавляем к новому родителю
                } else {
                    this.items.push(this.editingItem); // Если ничего не выбрано, добавляем в общий массив
                }
            }

            // Считаем общую стоимость для родительского элемента
            const updateParentCost = (item) => {
                if (item.children) {
                    item.price = item.children.reduce((total, child) => total + (child.price * child.quantity), 0);
                }
            };

            // Обновляем стоимость для всех родительских элементов
            this.items.forEach(item => {
                updateParentCost(item);
            });

            this.saveItems();
            this.editingItem = null;
        },
        removeItem(item) {
            const findAndRemove = (array, itemToRemove) => {
                const index = array.findIndex(item => item.name === itemToRemove.name);
                if (index !== -1) {
                    array.splice(index, 1);
                } else {
                    for (const child of array) {
                        if (child.children) {
                            findAndRemove(child.children, itemToRemove);
                        }
                    }
                }
            };

            findAndRemove(this.items, item);
            this.saveItems();
        },
        closeEditModal() {
            this.editingItem = null;
        },
        openAddProductModal() {
            this.isAddProductModalOpen = true;
            this.newItem = { name: '', price: null, quantity: null, children: [] };
        },
        closeAddProductModal() {
            this.isAddProductModalOpen = false;
            this.newItem = { name: '', price: null, quantity: null, children: [] };
        },
        validateQuantity() {
            if (this.editingItem.quantity < 0) {
                alert('Количество не может быть отрицательным');
                this.editingItem.quantity = 0;
            }
        },
        handleSelection(selected) {
            this.selectedItem = selected;
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
