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
        <select v-model="parentItem" @change="updateParentItem">
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
          </tr>
        </thead>
        <tbody>
          <template v-for="item in itemes" :key="item.name">
            <tr>
              <td>{{ item.name }}</td>
              <td>{{ item.price }}</td>
              <td>{{ item.quantity }}</td>
              <td>{{ item.price * item.quantity }}</td>
              <td>
                <button @click="removeItem(item)">Удалить</button>
                <button @click="addQuantity(item)">Добавить</button>
              </td>
            </tr>
            <template v-for="child in item.children" :key="child.name">
              <tr>
                <td>{{ child.name }}</td>
                <td>{{ child.price }}</td>
                <td>{{ child.quantity }}</td>
                <td>{{ child.price * child.quantity }}</td>
                <td>
                  <button @click="removeItem(child)">Удалить</button>
                  <button @click="addQuantity(child)">Добавить</button>
                </td>
              </tr>
              <template v-for="childer in child.children" :key="childer.name">
                <tr>
                  <td>{{ childer.name }}</td>
                  <td>{{ childer.price }}</td>
                  <td>{{ childer.quantity }}</td>
                  <td>{{ childer.price * childer.quantity }}</td>
                  <td>
                    <button @click="removeItem(childer)">Удалить</button>
                    <button @click="addQuantity(childer)">Добавить</button>
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
      parentItem: null
    };
  },
  methods: {
    // Загрузка данных из localStorage
    loadItems() {
      const savedItems = localStorage.getItem('itemes');
      return savedItems ? JSON.parse(savedItems) : [
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

    // Сохранение данных в localStorage
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
      this.saveItems(); // Сохраняем данные после пересчета
    },

    addQuantity(item) {
      item.quantity += 1;
      this.sumItems();
    },

    removeItem(item) {
      if (item.quantity > 1) {
        item.quantity -= 1;
        this.sumItems();
      }
    },
    
    addItem() {
      const newItem = {
        name: this.newItemName,
        price: this.newItemPrice,
        quantity: this.newItemQuantity,
        children: []
      };

      if (this.parentItem) {
        this.parentItem.children.push(newItem);
      } else {
        this.itemes.push(newItem);
      }

      // Сброс значений формы
      this.newItemName = '';
      this.newItemPrice = null;
      this.newItemQuantity = null;
      this.parentItem = null;

      this.sumItems();
    },

    updateParentItem() {
      // Обновление выбранного родительского элемента
    },

    exportToExcel() {
      const data = [];
      for (const item of this.itemes) {
        data.push({ "name": item.name, "price": item.price, "quantity": item.quantity, "total": item.price * item.quantity });
        if (item.children) {
          for (const child of item.children) {
            data.push({ "name": child.name, "price": child.price, "quantity": child.quantity, "total": child.price * child.quantity });
            if (child.children) {
              for (const childish of child.children) {
                data.push({ "name": childish.name, "price": childish.price, "quantity": childish.quantity, "total": childish.price * childish.quantity });
              }
            }
          }
        }
      }

      const worksheet = XLSX.utils.json_to_sheet(data);
      const workbook = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(workbook, worksheet, "Товары");

      XLSX.writeFile(workbook, 'table_data.xlsx');
    }
  },
  created() {
    this.sumItems(); // Пересчитываем стоимость
  }
};
</script>


<style>
* {
  margin: 10px;
  padding: 10px;
  box-sizing: border-box;
}

body {
  font-family: 'Arial', sans-serif;
  line-height: 1.6;
  background-color: #f0f4f8; /* Светлый фон для большей контрастности */
  color: #343a40;
}

header {
  background-color: #007bff; /* Яркий цвет для заголовка */
  color: #fff;
  padding: 20px 0;
  text-align: center;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1); /* Тень для заголовка */
}

header h1 {
  margin-bottom: 10px;
  font-size: 2.5em; /* Увеличенный размер шрифта */
}

nav ul {
  list-style: none;
  display: flex;
  justify-content: center;
  margin-top: 10px;
}

nav ul li {
  margin: 0 20px;
}

nav ul li a {
  color: #fff;
  text-decoration: none;
  font-size: 1.2em; /* Увеличенный размер шрифта для ссылок */
  padding: 5px 10px;
  border-radius: 5px; /* Закругленные углы для ссылок */
  transition: background-color 0.3s; /* Плавный переход */
}

nav ul li a:hover {
  background-color: rgba(255, 255, 255, 0.2); /* Эффект наведения */
}

.hero {
  background-color: #343a40;
  color: white;
  padding: 80px 20px; /* Увеличенное внутреннее пространство */
  text-align: center;
}

.hero h2 {
  font-size: 2.5em; /* Увеличенный размер заголовка */
}

.btn {
  background-color: #28a745; /* Зелёный цвет для кнопок */
  color: #fff;
  padding: 15px 30px; /* Увеличенное внутреннее пространство */
  text-decoration: none;
  border-radius: 5px;
  font-size: 1.1em; /* Увеличенный размер шрифта для кнопок */
  transition: background-color 0.3s; /* Плавный переход */
}

.btn:hover {
  background-color: #218838; /* Тёмно-зелёный цвет при наведении */
}

.products {
  padding: 40px 20px;
}

.add-product-form {
  display: flex;
  flex-direction: column;
  margin-bottom: 20px;
  background-color: #ffffff; /* Белый фон для формы */
  padding: 20px;
  border-radius: 5px; /* Закругленные углы */
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1); /* Тень для формы */
}

.add-product-form input,
.add-product-form select {
  margin-bottom: 15px;
  padding: 15px;
  border: 1px solid #ced4da;
  border-radius: 5px;
  font-size: 1em; /* Размер шрифта для вводимых полей */
}

.add-btn,
.export-btn {
  background-color: #007bff; /* Яркий цвет для кнопок */
  color: #fff;
  padding: 12px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 1.1em; /* Увеличенный размер шрифта для кнопок */
  transition: background-color 0.3s; /* Плавный переход */
}

.add-btn:hover,
.export-btn:hover {
  background-color: #0056b3; /* Тёмно-синий цвет при наведении */
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
  background-color: #ffffff; /* Белый фон для таблицы */
  border-radius: 5px; /* Закругленные углы */
  overflow: hidden; /* Убирает скругления от таблицы */
}

table th,
table td {
  border: 1px solid #dee2e6;
  padding: 15px;
  text-align: left;
}

table th {
  background-color: #f8f9fa; /* Цвет заголовка таблицы */
  font-weight: bold;
  font-size: 1.1em; /* Увеличенный размер шрифта для заголовка */
}

.contact {
  background-color: #007bff; /* Цвет фона для контактов */
  color: white;
  padding: 40px 20px;
  text-align: center;
}

.contact h2 {
  margin-bottom: 20px; /* Промежуток между заголовком и текстом */
  font-size: 2em; /* Увеличенный размер заголовка */
}

footer {
  background-color: #343a40; /* Цвет фона для подвала */
  color: white;
  text-align: center;
  padding: 15px 0;
  position: relative; /* Позволяет подвалу оставаться на месте */
}

footer p {
  margin: 0; /* Убирает отступы для текста в подвале */
  font-size: 0.9em; /* Размер шрифта для текста в подвале */
}

/* Мобильная адаптация */
@media (max-width: 768px) {
  nav ul {
    flex-direction: column; /* Вертикальная ориентация меню */
  }

  nav ul li {
    margin: 10px 0; /* Увеличенный отступ между элементами */
  }

  .add-product-form {
    margin: 0 10px; /* Отступы для формы на мобильных устройствах */
  }
}

</style>
