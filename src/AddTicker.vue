<template>
    <section>
        <div class="cryptonomicon__group">
            <div class="cryptonomicon__row">
                <label 
                for="wallet" 
                class="cryptonomicon__label"
                >Тикер</label
                >

                <input
                v-model="ticker"
                @keydown.enter="add"
                type="text"
                name="wallet"
                id="wallet"
                class="cryptonomicon__input"
                placeholder="Например BTC"
                />
            </div>
        </div>

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
    }
  },

  emits: {
    "add-ticker": value => typeof value === "string" && value.length > 0
  },

  data() {
    return { ticker: "" };
  },

  methods: {
    add() {
      if (this.ticker.length === 0) {
        return;
      }
      this.$emit("add-ticker", this.ticker);
      this.ticker = "";
    }
  }
};
</script>

<style scoped>
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
</style>