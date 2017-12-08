<template>
  <div>
    <table>
      <tr v-for="ligne in grille">
        <td v-for="cellule in ligne"
          :class="cellule.clique ? 'red' : 'green'"
          @click="click(cellule)">{{cellule.bombe}}</td>
      </tr>
    </table>
  </div>
</template>

<script>
  export default {
    name: 'App',
    data () {
      const grille = []
      // Lignes
      for(var i = 0; i<10; i++){
      	// Cellules
      	var ligne = []
      	for(var j = 0; j<10; j++){
      		ligne.push({
      			bombe: true,
      			x: j,
      			y: i,
      			clique: false
      		})
      	}
      	grille.push(ligne)

      }
      return {
      	grille: grille
      }
    },
    methods: {
    	click(cellule){
    		console.log(`x: ${cellule.x} - y:${cellule.y}`)
    		cellule.clique = true
    	}
    }
  }
</script>

<style>
	.red{
		background-color: red
	}
	.green{
		background-color: green;
	}
</style>