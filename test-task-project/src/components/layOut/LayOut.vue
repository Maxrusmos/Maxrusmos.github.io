<template>
  <div class="main__container">
    <div class="calculation__container">
      <div class="inputs__container">
        <LayOutInputsLabels
          v-model="price"
          :id="'price'"
          :placeholder="'Цена'"
          @update:modelValue="logInputEvent('price', $event)"
        ></LayOutInputsLabels>
        <LayOutInputsLabels
          v-model="quantity"
          :id="'quantity'"
          :placeholder="'Количество'"
          @update:modelValue="logInputEvent('quantity', $event)"
        ></LayOutInputsLabels>
        <LayOutInputsLabels
          v-model="sum"
          :id="'sum'"
          :placeholder="'Сумма'"
          @update:modelValue="logInputEvent('sum', $event)"
        ></LayOutInputsLabels>
        <div class="button__container">
          <button @click="saveToLocalStorage">Отправить</button>
          <div class="localStoreDisplay">
            <label v-html="jsonDataWithLineBreaks"></label>
          </div>
        </div>
      </div>
    </div>
    <div class="info__container">
      <div v-for="event in events" :key="event.timestamp" class="event">
        {{ event.timestamp }} - {{ event.message }}
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import {
  defineComponent,
  ref,
  onMounted,
  watch,
  reactive,
  computed,
} from "vue";
import LayOutInputsLabels from "../layOutInputsLabels/LayOutInputsLabels.vue";

type Field = "price" | "quantity" | "sum";

export default defineComponent({
  setup() {
    const events = ref([] as Array<{ timestamp: string; message: string }>);
    let backendTimer: number | null = null;

    const price = ref(0);
    const quantity = ref(0);
    const sum = ref(0);
    let buttonClickCount = 0;

    const savedPrice = ref("");
    const savedQuantity = ref("");
    const savedSum = ref("");
    const savedCounter = ref("");

    const fieldPriority = reactive({
      price: 0,
      quantity: 0,
      sum: 0,
    });

    const updateFieldPriority = (field: Field) => {
      fieldPriority[field] = Date.now();
    };

    const jsonData = computed(() => {
      return JSON.stringify(
        {
          price: savedPrice.value,
          qty: savedQuantity.value,
          amount: savedSum.value,
          counter: savedCounter.value,
        },
        null,
        2
      );
    });

    const jsonDataWithLineBreaks = computed(() => {
      return jsonData.value.replace(/\n/g, "<br>").replace(/\s/g, "&nbsp;");
    });

    watch([price, quantity, sum], ([newPrice, newQuantity, newSum]) => {
      const latestField = Object.keys(fieldPriority).reduce((a, b) =>
        fieldPriority[a as keyof typeof fieldPriority] >
        fieldPriority[b as keyof typeof fieldPriority]
          ? a
          : b
      );

      if (
        latestField === "price" &&
        newPrice !== null &&
        newQuantity !== null
      ) {
        sum.value = parseFloat((newPrice * newQuantity).toFixed(2));
      } else if (
        latestField === "quantity" &&
        newPrice !== null &&
        newQuantity !== null
      ) {
        sum.value = parseFloat((newPrice * newQuantity).toFixed(2));
      } else if (latestField === "sum" && newSum !== null) {
        if (newPrice !== null) {
          quantity.value = parseFloat((newSum / newPrice).toFixed(2));
        } else if (newQuantity !== null) {
          price.value = parseFloat((newSum / newQuantity).toFixed(2));
        }
      }
    });

    onMounted(() => {
      const storedCount = localStorage.getItem("buttonClickCount");
      if (storedCount) {
        buttonClickCount = parseInt(storedCount);
      }
      price.value = parseFloat(localStorage.getItem("price")!);
      quantity.value = parseFloat(localStorage.getItem("quantity")!);
      sum.value = parseFloat(localStorage.getItem("sum")!);

      loadFromLocalStorage();
    });

    const isEven = (number: number) => number % 2 === 0;

    const saveToLocalStorage = () => {
      if (backendTimer) {
        clearTimeout(backendTimer);
      }
      backendTimer = setTimeout(() => {
        logButtonClickEvent();
        let dataIsGone = false;
        if (isEven(+sum.value)) {
          localStorage.setItem("price", price.value.toString());
          localStorage.setItem("quantity", quantity.value.toString());
          localStorage.setItem("sum", sum.value.toString());
          dataIsGone = true;
        }
        updateLocalStorageValues();
        logResult(dataIsGone ? { success: true } : { success: false });
      }, 1000);

      buttonClickCount += 1;
      localStorage.setItem("counter", buttonClickCount.toString());
    };

    const loadFromLocalStorage = () => {
      const savedPriceValue = localStorage.getItem("price");
      const savedQuantityValue = localStorage.getItem("quantity");
      const savedSumValue = localStorage.getItem("sum");
      const savedCounterValue = localStorage.getItem("counter");

      savedPrice.value = savedPriceValue ?? "";
      savedQuantity.value = savedQuantityValue ?? "";
      savedSum.value = savedSumValue ?? "";
      savedCounter.value = savedCounterValue ?? "";
    };

    const updateLocalStorageValues = () => {
      loadFromLocalStorage();
    };

    const logEvent = (message: string) => {
      const timestamp = new Date().toISOString();
      events.value.unshift({ timestamp, message });
    };

    const logResult = (success: any) => {
      loadFromLocalStorage();
      logEvent(
        `Ответ: success: ${success.success} || В localStorage: цена: ${savedPrice.value}, количество: ${savedQuantity.value}, сумма: ${savedSum.value}`
      );
    };

    const logButtonClickEvent = () => {
      loadFromLocalStorage();
      logEvent(
        `Кнопка ОТПРАВИТЬ была нажата. Вы отправили: цена: ${price.value}, количество: ${quantity.value}, сумма: ${sum.value} || В localStorage было: цена: ${savedPrice.value}, количество: ${savedQuantity.value}, сумма: ${savedSum.value}`
      );
    };

    const logInputEvent = (inputId: Field, value: string) => {
      if (/^\d.+$/.test(value)) {
        logEvent(`Поле ${inputId} изменилось: ${value}`);
      }
      updateFieldPriority(inputId);
    };
    return {
      saveToLocalStorage,
      price,
      quantity,
      sum,
      loadFromLocalStorage,
      logInputEvent,
      events,
      savedPrice,
      savedQuantity,
      savedSum,
      savedCounter,
      jsonDataWithLineBreaks,
    };
  },
  components: {
    LayOutInputsLabels,
  },
});
</script>

<style scoped>
.main__container {
  display: flex;
  flex-direction: column;
  gap: 16px;
  justify-content: center;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 70%;
  height: 70vh;
}

.inputs__container {
  display: flex;
  justify-content: space-around;
  gap: 16px;
}

.button__container {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 16px;
  > button {
    cursor: pointer;
  }
}

.localStoreDisplay {
  display: flex;
  flex-direction: column;
  gap: 12px;
  border: 1px solid black;
  padding: 12px;
  box-sizing: border-box;
}

.info__container {
  height: 100%;
  border: 1px solid black;
  padding: 12px;
  box-sizing: border-box;
}
</style>
