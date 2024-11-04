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
            <div>
        <button @click="exportToExcel" class="export-btn">Экспортировать в Excel</button>
      </div>

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
                        <RecursiveTr :data="item" :level="0" :items="items" @edit="editItem" @remove="removeItem" />
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

                <RecursiveSelect :items="items" :editItem="editingItem" @select="handleParentSelection" />
                
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
import * as XLSX from 'xlsx';
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
            originalParent: null, // Хранит оригинального родителя для корректного переноса
            exportItems: []
        };
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

            const newItem = { ...this.newItem, children: [] };

            if (this.selectedItem) {
                const target = this.findItemByName(this.items, this.selectedItem[this.selectedItem.length - 1]);
                target.children.push(newItem);
            } else {
                this.items.push(newItem);
            }

            this.updatePrices();
            this.saveItems();
            this.closeAddProductModal();
        },
        updatePrices() {
            const calculatePrice = (item) => {
                if (item.children && item.children.length > 0) {
                    item.price = item.children.reduce(
                        (total, child) => total + child.price * child.quantity,
                        0
                    );
                }
            };

            const updateItemPrices = (items) => {
                items.forEach((item) => {
                    if (item.children) {
                        updateItemPrices(item.children);
                    }
                    calculatePrice(item);
                });
            };

            updateItemPrices(this.items);
        },
        openAddProductModal() {
            this.newItem = { name: '', price: null, quantity: null, children: [] };
            this.isAddProductModalOpen = true;
        },
        closeAddProductModal() {
            this.isAddProductModalOpen = false;
        },
        editItem(item) {
            this.editingItem = { ...item }; // Создаем копию редактируемого товара
            this.selectedItem = item; // сохраняем текущий товар для редактирования
            this.newParentItem = null; // Сбрасываем выбранный родительский элемент
            this.originalParent = this.findParentOfItem(item.name); // Запоминаем оригинального родителя
        },
        removeItem(item) {
            const removeFromArray = (array, name) => {
                for (let i = 0; i < array.length; i++) {
                    if (array[i].name === name) {
                        array.splice(i, 1);
                        return true;
                    }
                    if (array[i].children) {
                        const found = removeFromArray(array[i].children, name);
                        if (found) return true;
                    }
                }
                return false;
            };

            removeFromArray(this.items, item.name);
            this.updatePrices();
            this.saveItems();
        },
        validateQuantity() {
            if (this.editingItem.quantity < 0) {
                this.editingItem.quantity = 0;
            }
        },
        handleSelection(selectedItem) {
            this.selectedItem = selectedItem;
        },
        handleParentSelection(selectedItem) {
            this.newParentItem = selectedItem;
        },
        findParentOfItem(name) {
            for (const item of this.items) {
                if (item.children) {
                    for (const child of item.children) {
                        if (child.name === name) {
                            return item;
                        }
                    }
                }
            }
            return null;
        },
       saveEdit() {
            const target = this.findItemByName(this.items, this.editingItem.name);
            if (target) {
                target.name = this.editingItem.name;
                target.price = this.editingItem.price;
                target.quantity = this.editingItem.quantity;

                // Перемещаем товар под новый родительский элемент, если выбран новый
                if (this.newParentItem && this.newParentItem.name !== target.name) {
                    // Удаляем товар из оригинального родителя
                    // this.originalParent.children = this.originalParent.children.filter(child => child.name !== target.name);
                    this.removeItem(this.editingItem);
                    // Добавляем товар под новый родительский элемент
                    const parentItem = this.findItemByName(this.items, this.newParentItem[this.newParentItem.length - 1]);
                    if (parentItem) {
                        parentItem.children.push(target);
                    }
                }
            }

            this.updatePrices();
            this.saveItems();
            this.closeEditModal();
        },
        closeEditModal() {
            this.editingItem = null;
            this.newParentItem = null; // сбрасываем выбранный родительский элемент
            this.selectedItem = null; // сбрасываем выбранный товар
            this.originalParent = null; // сбрасываем оригинального родителя
        },
        exportToExcel() {
            this.exportItems = [];
            this.exportArray(this.items);

            const worksheet = XLSX.utils.json_to_sheet(this.exportItems);
            const workbook = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(workbook, worksheet, "Товары");

            XLSX.writeFile(workbook, 'table_data.xlsx');
        },
        exportArray(items) {
            for (const item of items) {
                const d = {
                    name: item.name,
                    quantity: item.quantity,
                    price: item.price
                }
                this.exportItems.push(d)
                if (item.children && item.children.length > 0) {
                    this.exportArray(item.children)
                }
            }
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