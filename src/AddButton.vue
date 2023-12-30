<template>
    <button
        type="button"
        :disabled="disabled"
        class="cryptonomicon__button"
        :class="{
            'cryptonomicon__button--disablet' : disabled
        }"
    >
    <svg
        class="cryptonomicon__svg"
        xmlns="http://www.w3.org/2000/svg"
        width="30"
        height="30"
        viewBox="0 0 24 24"
        fill="#ffffff"
    >
        <path
        d="M13 7h-2v4H7v2h4v4h2v-4h4v-2h-4V7zm-1-5C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8z"
        ></path>
    </svg>
    Добавить
    </button>
</template>

<script>
export default {
  props: {
    disabled: {
      type: Boolean,
      required: false,
      default: false
    }
  }
};
</script>


<style scoped>
    .cryptonomicon__button {
    margin-top: 10px;
    padding: 10px;
    max-width: 200px;
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 10px;
    font-size: 20px;
    border-radius: 10px;
    border: 1px solid black;
    cursor: pointer;
    color: black;
    background-color: transparent;
    transition: all 0.3s ease-in-out;
}
.cryptonomicon__button--disablet {
    opacity: 0.3;
}
.cryptonomicon__button:hover {
    color: blue;
    border: 1px solid blue;
}
.cryptonomicon__button:hover .cryptonomicon__svg {
    fill: blue;
}
.cryptonomicon__svg {
    fill: black;
    transition: all 0.3s ease-in-out;
}
</style>