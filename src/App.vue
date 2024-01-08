<template>
  <div class="cryptonomicon">
    <div class="cryptonomicon__container container">
      
      <add-ticker 
          @add-ticker="add"
          :disabled="tooManyTickersAdded"
          :has-ticker="hasTicker"
          @focus="hasTicker = false"
      />

      <template v-if="tickers.length">
          <div class="cryptonomicon__filter filter-cryptonomicon">
              <hr class="cryptonomicon__border"/>
              <div class="filter-cryptonomicon__row">
                  <h2 class="filter-cryptonomicon__title">Фильтр:</h2>

                  <input 
                  v-model="filter"
                  class="filter-cryptonomicon__input" 
                  type="text"
                  >
              </div>

              <div class="filter-cryptonomicon__buttons">
                  <button 
                  class="filter-cryptonomicon__button"
                  v-if="page > 1"
                  @click="page = page - 1"
                  >
                      Назад
                  </button>

                  <button 
                  class="filter-cryptonomicon__button"
                  v-if="hasNextPage"
                  @click="page = page + 1"
                  >
                      Вперед
                  </button>
              </div>  
          </div>
        <hr class="cryptonomicon__border"/>
        <dl class="cryptonomicon__items">
          <div
            v-for="t in paginatedTickers"
            :key="t.name"
            @click="select(t)"
            :class="{
              'cryptonomicon__item--target': selectedTicker === t
            }"
            class="cryptonomicon__item"
          >
            <div class="cryptonomicon__text">
              <dt class="cryptonomicon__name">
                {{ t.name }} - USD
              </dt>
              <dd class="cryptonomicon__price">
                {{ formatPrice(t.price) }}
              </dd>
            </div>

            <button
              @click.stop="handleDelete(t)"
              class="cryptonomicon-button__delete"
            >
              <svg
                class="cryptonomicon-button__delete--svg"
                xmlns="http://www.w3.org/2000/svg"
                viewBox="0 0 20 20"
                fill="#718096"
                aria-hidden="true"
              >
                <path
                  fill-rule="evenodd"
                  d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z"
                  clip-rule="evenodd"
                ></path>
                </svg>
                Удалить
            </button>
          </div>
        </dl>
        <hr class="cryptonomicon__border" />
      </template>

      <section v-if="selectedTicker" class="cryptonomico-graph">
        <h3 class="cryptonomico-graph__title">
          {{ selectedTicker.name }} - USD
        </h3>
        <div class="cryptonomico-graph__items" ref="graph">
          <div 
          v-for="(bar, index) in normalizedGraph"
          :key="index"
          :style="{ height: `${bar}%` }"
          class="cryptonomico-graph__item"
          ></div>
        </div>
        <button
          @click="selectedTicker = null"
          type="button"
          class="cryptonomico-graph__button"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            xmlns:xlink="http://www.w3.org/1999/xlink"
            xmlns:svgjs="http://svgjs.com/svgjs"
            version="1.1"
            width="30"
            height="30"
            x="0"
            y="0"
            viewBox="0 0 511.76 511.76"
            style="enable-background:new 0 0 512 512"
            xml:space="preserve"
          >
            <g>
              <path
                d="M436.896,74.869c-99.84-99.819-262.208-99.819-362.048,0c-99.797,99.819-99.797,262.229,0,362.048    c49.92,49.899,115.477,74.837,181.035,74.837s131.093-24.939,181.013-74.837C536.715,337.099,536.715,174.688,436.896,74.869z     M361.461,331.317c8.341,8.341,8.341,21.824,0,30.165c-4.16,4.16-9.621,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    l-75.413-75.435l-75.392,75.413c-4.181,4.16-9.643,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    c-8.341-8.341-8.341-21.845,0-30.165l75.392-75.413l-75.413-75.413c-8.341-8.341-8.341-21.845,0-30.165    c8.32-8.341,21.824-8.341,30.165,0l75.413,75.413l75.413-75.413c8.341-8.341,21.824-8.341,30.165,0    c8.341,8.32,8.341,21.824,0,30.165l-75.413,75.413L361.461,331.317z"
                fill="#718096"
                data-original="#000000"
              ></path>
            </g>
          </svg>
        </button>
      </section>
    </div>
  </div>
</template>

<script>
import { subscribeToTicker, unsubscribeFromTicker } from "./api";
import AddTicker from "./AddTicker.vue"

export default {
name: "App",

  components: {
      AddTicker,
  },


data() {
  return {
      hasTicker: false,

  // // Ввод данных в инпут Тикер 
  //   ticker: "",
  
    // Ввод данных в инпут Фильтр 
    filter: '',

  // массив тикеров
    tickers: [],

  // Текущий тикер
  selectedTicker: null,

  // данные состояния. График.
    graph: [],
    maxGrapgElements: 1,

    // текущая страница
    page: 1,
  };
},

// метод жизниного цикла
created() {
  // Грузим данные из 'Урла'
  const windowData = Object.fromEntries(
    new URL(window.location).searchParams.entries()
  );

  const VALID_KEYS = ["filter", "page"];

  VALID_KEYS.forEach(key => {
    if (windowData[key]) {
      this[key] = windowData[key];
    }
  });


  // if (windowData.filter) {
  //   this.filter = windowData.filter;
  // }

  // if (windowData.page) {
  //   this.page = windowData.page;
  // }

  // Грузим информацию в LocalStorage
  const tickersData = localStorage.getItem('cryptonomicon-list');
  if (tickersData) {
    this.tickers = JSON.parse(tickersData);
    this.tickers.forEach(ticker => {
      subscribeToTicker(ticker.name, newPrice =>
        this.updateTicker(ticker.name, newPrice)
      );
    });
  }

  setInterval(this.updateTickers, 5000);
},

mounted() {
  window.addEventListener("resize", this.calculateMaxGraphElements);
},

beforeUnmount() {
  window.removeEventListener("resize", this.calculateMaxGraphElements);
},


computed: {
  suggestions() {
      return console.log(this.tickers)
  },

  tooManyTickersAdded() {
    return this.tickers.length > 24;
  },

  // фильтрация = const start = (this.page  - 1) * 8;
  startIndex() {
      return (this.page - 1) * 8;
  },

  // фильтрация = const end = this.page * 8;
  endIndex() {
      return this.page * 8;
  },

   // ищем все тикеры, которые подходят нашему введенному фильтру
   filteredTickers() {
     return this.tickers.filter(ticker => ticker.name.includes(this.filter));
  },

  //
  paginatedTickers() {
      return this.filteredTickers.slice(this.startIndex, this.endIndex);
  },

  // this.hasNextPage = filteredTickers.length > end;
  hasNextPage() {
      return this.filteredTickers.length > this.endIndex;
  },

  // график
  normalizedGraph() {
      const maxValue = Math.max(...this.graph);
      const minValue = Math.min(...this.graph);

      if (maxValue === minValue) {
          return this.graph.map(() => 50);
      }

      return this.graph.map(
          price => 5 + ((price - minValue) * 95) / (maxValue - minValue)
      );
  },

  //
  pageStateOptions() {
      return {
          filter: this.filter,
          page: this.page,
      };
  },
},

methods: {

  calculateMaxGraphElements() {
    if (!this.$refs.graph) {
      return;
    }

    this.maxGraphElements = this.$refs.graph.clientWidth / 38;
  },


  // Получаем данные с сервера
  updateTicker(tickerName, price) {
    this.tickers.filter(t => t.name === tickerName).forEach(t => {
        if (t === this.selectedTicker) {
          this.graph.push(price);

          while (this.graph.length > this.maxGraphElements) {
            this.graph.shift();
          }

        }
        
        t.price = price;
      });
  },

  formatPrice(price) {
    if (price === "-") {
      return price;
    }
    return price > 1 ? price.toFixed(2) : price.toPrecision(2);
  },
  
  // добавление тикера
  add(ticker) {
    const currentTicker = {
      name: ticker.toUpperCase(),
      price: "-"
    };

    if (!this.tickers.some(t => t.name === currentTicker.name)) {
      this.tickers = [...this.tickers, currentTicker];
      // console.log(this.tickers)
      this.hasTicker = false;
    } else {
      this.hasTicker = true;
    }

    this.filter = '';
    subscribeToTicker(currentTicker.name, newPrice =>
      this.updateTicker(currentTicker.name, newPrice)
    );
  },

  // Выбор тикер и его очищение при смены таргета
  select(ticker) {
      this.selectedTicker = ticker;
  },

  // Удаление тикера
  handleDelete(tickerToRemove) {
    this.tickers = this.tickers.filter(t => t !== tickerToRemove);

  // При удаление тикера удаляем график
    if (this.selectedTicker === tickerToRemove) {
      this.selectedTicker = null;
    }
    unsubscribeFromTicker(tickerToRemove.name);
  },
},

  // реагируем на изминения переменных с помощью watch
  watch: {
      // Когда изменяется выбранный тикер, то сбрасываем график
      selectedTicker() {
          this.graph = [];

          this.$nextTick().then(this.calculateMaxGraphElements);
      },

      // когда меняются тикеры, сохранять в LocalStorage
      tickers() {
          localStorage.setItem('cryptonomicon-list', JSON.stringify(this.tickers));
      },

      // Если на странице нету тикеров, то сбросить страницу назад(Например, страница 3, удаляем все элементы там и переходим на страницу 2)
      paginatedTickers() {
          if (this.paginatedTickers.length === 0 && this.page > 1) {
              this.page -= 1;
          }
      },


      // если изменился фильтр, обновляет URL страницы
      filter() {
          this.page = 1;
      },

      // если изменилась страничка, обновляет URL страницы
      pageStateOptions(value) {
          window.history.pushState(
          null,
          document.title,
          `${window.location.pathname}?filter=${value.filter}&page=${value.page}`
          );
      }
  }
};

</script>

<style scoped>
.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 15px;
}
.filter-cryptonomicon__row {
    display: flex;
    align-items: center;
    gap: 10px;
}
.filter-cryptonomicon__title {
    font-size: 25px;
    color: black;
}
.filter-cryptonomicon__input {
    display: flex; 
    align-items: center; 
    justify-content: center;
    max-width: 150px;
    width: 100%;
    padding: 10px 20px;
    border: 1px solid black;
    outline-color: blue;
    border-radius: 10px;
    font-size: 16px;
    color: black;
}
.filter-cryptonomicon__buttons {
    display: flex;
    align-items: center;
    gap: 10px;
}
.filter-cryptonomicon__button {
    padding: 10px;
    max-width: 150px;
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 10px;
    font-size: 16px;
    border-radius: 10px;
    border: 1px solid black;
    cursor: pointer;
    color: black;
    background-color: transparent;
    transition: all 0.3s ease-in-out;
}
.filter-cryptonomicon__button:hover {
    color: blue;
    border: 1px solid blue;
}
.cryptonomicon__items {
    display: flex;
    align-items: flex-start;
    flex-wrap: wrap;
    column-gap: 40px;
    row-gap: 40px;
}

.cryptonomicon__item {
    max-width: 230px;
    width: 100%;
    text-align: center;
    padding: 20px;
    border-radius: 10px;
    cursor: pointer;
}
.cryptonomicon__item--target {
    border: 3px solid blue;
}
.cryptonomicon__text {
    text-align: center;
}
.cryptonomicon__name {
    font-size: 16px;
    color: gray;
    margin: 0;
}
.cryptonomicon__price {
    font-size: 20px;
    color: black;
    padding: 10px 0;
    margin: 0;
}
.cryptonomicon-button__delete {
    padding: 10px;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 10px;
    width: 100%;
    border-radius: 10px;
    border: none;
    cursor: pointer;
    background-color: transparent;
    transition: all 0.3s ease-in-out;
}
.cryptonomicon-button__delete:hover {
    color: blue;
}
.cryptonomicon-button__delete:hover .cryptonomicon-button__delete--svg {
    fill: blue;
}
.cryptonomicon-button__delete--svg {
    width: 20px;
    height: 20px;
    transition: all 0.3s ease-in-out;
}
.cryptonomico-graph {
    position: relative;
}
.cryptonomico-graph__title {
    margin-top: 2rem;
    margin-bottom: 2rem;
    --text-opacity: 1;
    color: #1a202c;
    color: rgba(26, 32, 44, var(--text-opacity));
    font-size: 1.125rem;
}
.cryptonomico-graph__items {
    display: flex;
    align-items: flex-end;
    height: 16rem;
    border-left-width: 1px;
    border-bottom-width: 1px;
    --border-opacity: 1;
    border-color: #718096;
    border-color: rgba(113, 128, 150, var(--border-opacity));

}
.cryptonomico-graph__item {
    width: 2.5rem;
    height: 6rem;
    --bg-opacity: 1;
    background-color: blue;
    background-color: rgba(0, 0, 153, var(--bg-opacity));
    border-width: 1px;
    
}
.cryptonomico-graph__button {
    position: absolute;
    top: 0;
    right: 0;
    background-color: transparent;
    background-image: none;
    cursor: pointer;
}
*, ::before, ::after {
    box-sizing: border-box;
    border-width: 0;
    border-style: solid;
    border-color: #e2e8f0;
}
.cryptonomicon__border {
    border: 1px solid rgb(188 187 187);
}
@media (max-width: 480px) {
    .cryptonomicon__items {
        justify-content: center;
    }
  }
</style>