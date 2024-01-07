<template>
  <section>
      <div class="cryptonomicon__group">
          <div class="cryptonomicon__row">
              <label 
                  for="wallet" 
                  class="cryptonomicon__label"
              >
                  Тикер
              </label>

              <input
                  v-model="ticker"
                  @keydown.enter="add"
                  @focus="$emit('focus')"
                  
                  class="cryptonomicon__input"
                  placeholder="Например BTC"
              />
          </div>
          <ul v-if="matched.length > 0" class="cryptonomicon__matched">
              <li v-for="symbol in matched" :key="symbol" @click="addCoin(symbol)">{{ symbol.Symbol }}</li>
          </ul>
      </div>

      <p 
          v-if="hasTicker"
          class="cryptonomicon__message"
      >
          Такой тикер уже существует
      </p>

      <add-button
          @click="add"
          :disabled="disabled"
      />
  </section>
</template>

<script>
import AddButton from "./AddButton.vue";

export default {
components: {
  AddButton
},

props: {
  disabled: {
    type: Boolean,
    required: false,
    default: false
  },
  hasTicker: {
      type: Boolean,
      required: false,
      default: false
  }
},

emits: [ "add-ticker", "focus", ],

data() {
  return {
      // Введенный текст в инпут
      ticker: "",

      // Массив для хранения найденных символов
      matchedSymbols: [],
      
      // массив тикеров из сетевого запроса
      allTickers: [],
  };
},


  // сетевой запрос тикеров
  async mounted() {
      const response = await fetch('https://min-api.cryptocompare.com/data/all/coinlist?summary=true');
      const result = await response.json();
      this.allTickers = Object.values(result.Data);
  },

  computed: {
      matched() {
          if (!this.ticker.trim()) {
              return [];
          }

          return this.allTickers.filter(coin => {
              return coin.Symbol.toLowerCase().startsWith(this.ticker.toLowerCase().trim()) || 
              coin.FullName.toLowerCase().startsWith(this.ticker.toLowerCase().trim());
          })
          
         .slice(0, 5);
      },
  },

methods: {
  add() {
    if (!this.ticker.trim()) {
      return;
    }
    this.$emit("add-ticker", this.ticker);
    this.ticker = "";
  },

  addCoin(symbol) {
      this.ticker = symbol.Symbol;
      this.add();
  }
},

};
</script>

<style scoped>
.cryptonomicon__message {
  font-style: 16px;
  color: red;
}
.cryptonomicon__row {
  width: 100%;
}
.cryptonomicon__label {
  font-size: 30px;
  color: black;
}
.cryptonomicon__input {
  margin-top: 10px;
  display: flex; 
  align-items: center; 
  justify-content: center;
  max-width: 300px;
  width: 100%;
  padding: 10px 20px;
  border: 1px solid black;
  outline-color: blue;
  border-radius: 10px;
  font-size: 20px;
  color: black;
}
.cryptonomicon__matched {
  display: flex;
  align-items: center;
  gap: 20px;
}
ul { list-style: none; }
</style>