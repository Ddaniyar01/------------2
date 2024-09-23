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
      <div class="product-list">
        <div class="product-item">
          <h3>Кузов</h3>

        </div>
        <div class="product-item">
          <h3>Двигатель</h3>

        </div>
        <div class="product-item">
          <h3>Салон</h3>

        </div>

      </div>
      <div>
    <button @click="exportToExcel">Экспортировать в Excel</button>
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
              <tr v-for="childer in child.children" :key="childer.name">
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
        </tbody>
      </table>
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
      itemes: [
        {
          name: "Кузов",
          price: null,
          quantity: 2,
          children: [
            {
              name: "Двери",
              price: null,
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
      price: null,
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
        },
        {
  name: "Салон",
  price: null,
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
      ]
    }
      ],
      

    };
  },
  created() {
    this.sumItems();
  },
  methods: {
    sumItems() {
      for (const item of this.itemes) {
        let price = 0;
        if (item.children) {
          for (const child of item.children) {
            let child_price = 0;
            if (child.children) {
              for (const childer of child.children) {
                child_price += childer.price * childer.quantity;
              }
              child.price = child_price;
            }
            price += child.price * child.quantity;
          }
        }
        item.price = price
      }
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
    exportToExcel() {
      // Данные для таблицы
      const data = [];
      for (const item of this.itemes) {
        data.push({ "name": item.name, "price": item.price, "quantity": item.quantity, "total": item.price * item.quantity })
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

      // Преобразуем данные в формат Excel
      const worksheet = XLSX.utils.json_to_sheet(data);
      const workbook = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(workbook, worksheet, "Товары");

      // Генерируем и загружаем файл
      XLSX.writeFile(workbook, 'table_data.xlsx');
    }
  }
};
</script>

<style>
* {
  margin: 10px;
  padding: 10px;
  box-sizing: border-box;
  color: black;
}

body {
  font-family: Arial, sans-serif;
  line-height: 1.6;
}

header {
  background-color: #333;
  color: #fff;
  padding: 10px 0;
  text-align: center;
}

header h1 {
  margin-bottom: 10px;
}

nav ul {
  list-style: none;
  display: flex;
  justify-content: center;
}

nav ul li {
  margin: 0 15px;
}

nav ul li a {
  color: #fff;
  text-decoration: none;
  font-size: 18px;
}

.hero {
  background-color: #f4f4f4;
  padding: 50px 0;
  text-align: center;
}

.hero h2 {
  font-size: 36px;
  margin-bottom: 10px;
}

.hero p {
  font-size: 18px;
  margin-bottom: 20px;
}

.btn {
  background-color: #333;
  color: #fff;
  padding: 10px 20px;
  text-decoration: none;
  border-radius: 5px;
  font-size: 16px;
}

.products {
  padding: 50px;
  background-color: #fff;
}

.products h2 {
  text-align: center;
  margin-bottom: 20px;
}

.product-list {
  display: flex;
  justify-content: space-around;
}

.product-item {
  background-color: #f9f9f9;
  padding: 20px;
  text-align: center;
  border: 1px solid #ddd;
  border-radius: 5px;
  width: 30%;
}

.product-item h3 {
  margin-bottom: 10px;


}
body {
  font-family: Arial, sans-serif;
  margin: 10px
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background-color: #f2f2f2;
}

th,
td {
  padding: 10px;
  border: 1px solid #ddd;
  text-align: left;
}

button {
  margin-right: 5px;
  padding: 5px 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 3px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}

.contact {
  padding: 50px;
  background-color: #f4f4f4;
  text-align: center;
}

footer {
  background-color: #333;
  color: #fff;
  padding: 20px 0;
  text-align: center;
  margin-top: 20px;
}
</style>