<template>
  <fieldset>
    <legend>
      TABLE
      <span v-show="totalAmount !== 0"
        >TOTAL: {{ totalAmount.toFixed(2) }} <span ref="currency"></span
      ></span>
      <input v-model="command" placeholder="input command" />
      <button @click="commandEnter()">Enter</button>
    </legend>

    <table>
      <thead>
        <tr>
          <th>Date</th>
          <th>Products</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in rows" :key="row">
          <td>{{ row.date }}</td>
          <td>
            <table>
              <tbody>
                <tr v-for="product in row.products" :key="product">
                  <td>
                    {{ product.name }}
                  </td>
                  <td>
                    {{ product.price }}
                  </td>
                  <td>
                    {{ product.currency }}
                  </td>
                </tr>
              </tbody>
            </table>
          </td>
        </tr>
      </tbody>
    </table>
  </fieldset>
</template>

<script>
import axios from "axios";
export default {
  name: "HelloWorld",
  data() {
    return {
      totalAmount: 0,
      rows: [],
      command: "",
    };
  },
  methods: {
    commandEnter() {
      const command = this.command;
      switch (command.split(" ")[0]) {
        case "add":
          this.addItem({
            date: command.split(" ")[1],
            price: command.split(" ")[2],
            currency: command.split(" ")[3],
            name: command.split(" ")[4],
          });
          break;
        case "list":
          this.sortByDate();
          break;
        case "clear":
          this.clearRowByDate({ date: command.split(" ")[1] });
          break;
        case "total":
          this.printTotalAmount({ currency: command.split(" ")[1] });
          break;
      }
      this.command = "";
    },

    addItem({ date = "", price = "", currency = "", name = "" }) {
      const index = this.rows.findIndex((object) => {
        return object.date === date;
      });

      if (index != -1)
        return this.rows[index].products.push({
          name: name,
          price: price,
          currency: currency,
        });

      this.rows.push({
        date: date,
        products: [{ name: name, price: price, currency: currency }],
      });
    },

    sortByDate() {
      this.rows = this.rows.sort((a, b) => {
        return new Date(a.date) - new Date(b.date);
      });
    },

    clearRowByDate({ date = "" }) {
      const index = this.rows.findIndex((object) => {
        return object.date === date;
      });

      if (index != -1) return this.rows.splice(index, 1);
    },

    async printTotalAmount({ currency = "" }) {
      this.totalAmount = 0;
      this.rows.forEach((row) => {
        row.products.forEach((product) => {
          this.caclulateCurrency({
            from: product.currency,
            to: currency,
            amount: product.price,
          }).then((res) => (this.totalAmount = this.totalAmount + res.result));
        });
      });
      this.$refs.currency.innerText = currency;
    },

    async caclulateCurrency({ from = "", to = "", amount = "" }) {
      try {
        const response = await axios.get(
          `https://api.apilayer.com/fixer/convert?to=${to}&from=${from}&amount=${amount}`,
          {
            headers: {
              apikey: "Jum5sYE2Z1uPTk36C4RvgQ6nEXWwO7Wf",
            },
          }
        );
        if (response.status == 200) {
          console.log(response.status);
        }
        return response.data;
      } catch (error) {
        console.log(error);
      }
    },
  },
};
</script>

<style scoped>
table {
  width: 100%;
  border: 1px solid;
}
input {
  width: 100%;
}
thead {
  background: grey;
}
tr {
  width: 100%;
}
</style>
