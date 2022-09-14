<template>
  <div class="drinks">
    <h1>Страница со списком напитков и рейтингами</h1>

    <table class="drinks-list">
      <thead>
         <tr>
           <th>Название</th>
           <th>Описание</th>
           <th>Рейтинг</th>
         </tr>
      </thead>
      <tbody>
         <tr v-for="drink in drinks" :key="drink.id">
           <td>{{ drink.name }}</td>
           <td>{{ drink.description }}</td>
           <td>{{ drink.rating }}/10</td>
         </tr>
      </tbody>

    </table>
  </div>
</template>

<style>
.drinks-list {
   border: 1px solid black;
   margin-left: auto;
   margin-right: auto;
}
.drinks-list > td {
   text-align: center;
}
</style>

<script>
export default {
  name: 'ListDrinks',
  components: {
  },
  data() {
    return {
      drinks: [
        {
          id: 1,
          name: 'вода',
          description: 'лучший способ утолить жажду',
          rating: 10,
        },
        {
          id: 2,
          name: 'компот',
          description: 'сладковато',
          rating: 4,
        },
      ],
    };
  },
};
</script>
