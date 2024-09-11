<template>
  <div class="list">
    <div class="sort-panel">
      Сортировка по рейтингу:
      <div class="buttons">
        <v-tooltip
          v-for="(button, index) in sortButtons"
          :key="index"
          :text="button.tooltip"
          location="top">
          <template v-slot:activator="{ props }">
            <v-btn
              v-bind="props"
              :icon="button.icon"
              density="compact"
              variant="tonal"
              class="sort-btn"
              color="blue"
              @click="sortList(button)" />
          </template>
        </v-tooltip>
      </div>
    </div>
    <section
      :style="`background: ${options.color}`"
      @drop="onDrop($event, options.id)"
      @dragover.prevent
      @dragenter.prevent>
      <div class="title">
        <h2>
          {{ options.title }}
        </h2>
        <div class="counter">
          <span>{{ cards.length }}</span>
        </div>
      </div>
      <v-btn
        icon="mdi-plus"
        variant="tonal"
        class="mt-5"
        color="white"
        @click="isNewCardDialogOpen = true" />

      <CardItem
        v-for="(card, index) in cards"
        draggable="true"
        :key="index"
        :card="card"
        :options="props.options"
        @delete-card="deleteCard(card.id)"
        @dragstart="onDragStart($event, card)" />

      <CardForm
        title="Добавление новой карточки"
        v-model="isNewCardDialogOpen"
        :form="form"
        @save-card="addCard"
        @close-form="isNewCardDialogOpen = false" />
    </section>
  </div>
</template>

<script setup>
import {ref, inject, watch} from 'vue';
  import CardItem from './CardItem.vue';
  import CardForm from './CardForm.vue';

  const firstList = inject('firstList');
  const secondList = inject('secondList');
  const lastList = inject('lastList');

  const props = defineProps({
    options: {},
  });

  const isNewCardDialogOpen = ref(false);

  const form = ref({
    image: '',
    id: null,
    title: '',
    description: '',
    category: '',
    price: null,
    rating: {},
  });

  let cards = ref([]);
  let cashedCards = ref([]);

  const sortButtons = ref([
    {
      tooltip: `По возрастанию`,
      action: `descending`,
      icon: `mdi-sort-descending`,
      sortMethod: function(a, b) {
        return a.rating.rate - b.rating.rate
      }
    },
    {
      tooltip: `По убыванию`,
      action: `ascending`,
      icon: `mdi-sort-ascending`,
      sortMethod: function(a, b) {
        return b.rating.rate - a.rating.rate
      }
    },
    {
      tooltip: `Без сортировки`,
      action: `default`,
      icon: `mdi-sort`,
      sortMethod: function() {
        return cashedCards.value;
      }
    },
  ]);
  watch([firstList, secondList, lastList], () => {
    getLocalCards();
  })
  function getLocalCards() {
    switch (props.options.id) {
      case 1:
        cashedCards.value = firstList.value;
        break;
      case 2:
        cashedCards.value = secondList.value;
        break;
      case 3:
        cashedCards.value = lastList.value;
        break;
    }
    cards.value = [...cashedCards.value];
  }
  getLocalCards();

  function addCard() {
    cards.value.unshift(form.value);
    isNewCardDialogOpen.value = false;
  }
  function deleteCard(id) {
    cards.value = cards.value.filter((card) => card.id != id);
  }
  function onDragStart(event, item) {
    event.dataTransfer.dropEffect = 'move';
    event.dataTransfer.effectAllowed = 'move';
    event.dataTransfer.setData('itemId', item.id.toString());
  }
  function onDrop(event, optionsId) {
    const itemId = parseInt(event.dataTransfer.getData('itemId'));
    let otherLists = [];

    switch (optionsId) {
      case 1:
        cards.value = firstList.value;
        otherLists = secondList.value.concat(lastList.value);
        break;
      case 2:
        cards.value = secondList.value;
        otherLists = firstList.value.concat(lastList.value);
        break;
      case 3:
        cards.value = lastList.value;
        otherLists = secondList.value.concat(firstList.value);
        break;
    }

    const newCard = otherLists.find((card) => card.id === itemId);

    if (newCard) {
      firstList.value = firstList.value.filter((card) => card.id !== newCard.id);
      secondList.value = secondList.value.filter((card) => card.id !== newCard.id);
      lastList.value = lastList.value.filter((card) => card.id !== newCard.id);
      cards.value.unshift(newCard);
    }

    switch (optionsId) {
      case 1:
        firstList.value = cards.value;
        break;
      case 2:
        secondList.value = cards.value;
        break;
      case 3:
        lastList.value = cards.value;
        break;
    }
  }
  function sortList(button) {
    getLocalCards();
    if (button.action !== 'default') {
      cards.value = [...cards.value].sort(button.sortMethod);
    } else {
      cards.value = cashedCards.value;
    }
  }
</script>

<style lang="scss" scoped>
.list {
  width: 400px;
  min-height: 500px;
  height: fit-content;
  .sort-panel {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
    margin-bottom: 10px;
    padding: 0 10px;
    color: white;
    .buttons {
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 5%;
      width: 40%;
    }
  }
  section {
    padding: 10px;
    width: 100%;
    min-height: 500px;
    height: fit-content;
    border-radius: 10px;
    display: flex;
    flex-direction: column;
    align-items: center;
    .title {
      padding-bottom: 10px;
      width: 90%;
      display: flex;
      align-items: center;
      justify-content: center;
      border-bottom: 2px solid white;
      .counter {
        margin-left: 10px;
        background: white;
        border-radius: 50%;
        width: 30px;
        height: 30px;
        display: flex;
        justify-content: center;
        align-items: center;
      }
    }
  }
}
</style>
