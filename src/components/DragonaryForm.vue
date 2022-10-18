<template>
  <div class="container">
    <div class="row">
      <h1 class="text-center">BREEDENATOR</h1>
    </div>
    <div class="row">
      <div class="col-md-2">
        <div class="card">
          <div class="card-header">Nuevo dragón</div>
          <div class="card-body">
            <label for="strength" class="form-label">Fuerza:</label>
            <input type="number" v-model="dragon.strength" class="form-control"/>
            <label for="cunning" class="form-label">Astucia:</label>
            <input type="number" v-model="dragon.cunning"  class="form-control"/>
            <label for="endurance" class="form-label">Dureza:</label>
            <input type="number" v-model="dragon.endurance"  class="form-control"/>
            <label for="endurance" class="form-label">Nombre:</label>
            <input type="text" v-model="dragon.name"  class="form-control"/>
            <label for="elements" class="form-label">Elemento:</label>
            <select name="elements" id="elements" class="form-select" v-model="dragon.element">
              <option v-for="element in elements" :key="element.id" :value="element.element">{{element.element}}</option>
            </select>
            <div class="d-grid">
              <button :disabled="dragons.length >= 2" class="btn btn-primary my-1" @click="addDragon">Guardar</button>
              <button class="btn btn-success my-1" @click="breedDragons">Breedear</button>
              <button class="btn btn-danger my-1" @click="clear">Borrar</button>
            </div>
          </div>
        </div>
      </div>
      <div class="col-md-10">
        <div class="card">
          <div class="card-header">Lista de Dragones</div>
          <div class="card-body">
            <b-table striped hover :items="dragons" show-empty small responsive>
              <template #cell(element)="item">
                <span :class="[item.item.element]">{{item.item.element}}</span>
              </template>
              <template #cell(type)="item">
                <span v-if="item.item.type === 1">Común</span>
                <span v-if="item.item.type === 2">Poco Común</span>
                <span v-if="item.item.type === 3">Raro</span>
              </template>
            </b-table>
          </div>
        </div>
        <div class="card">
          <div class="card-header">Generaciones</div>
          <div class="card-body">
            <b-table striped hover :items="breed_dragons" show-empty small responsive>
              <template #cell(element)="item">
                <span :class="[item.item.element]">{{item.item.element}}</span>
              </template>
              <template #cell(type)="item">
                <span v-if="item.item.type === 1">Común</span>
                <span v-if="item.item.type === 2">Poco Común</span>
                <span v-if="item.item.type === 3">Raro</span>
              </template>
            </b-table>
          </div>
        </div>
      </div>
    </div>
    
    <div class="row">
      <h2></h2>
      
    </div>
  </div>
</template>
<script>
export default {
  name: "DragonaryForm",
  data() {
      return {
        dragon: {
          name: '',
          strength: 0,
          cunning: 0,
          endurance: 0,
          element: null,
          total: 0,
          type: 0,
        },
        elements: [
          { id: 0, element:'Fire'},
          { id: 1, element:'Water'},
          { id: 2, element:'Earth'},
          { id: 3, element:'Wind'},
          { id: 4, element:'Storm'},
          { id: 5, element:'Ice'},
          { id: 6, element:'Plant'},
          { id: 7, element:'Poison'},
        ],
        dragons: [],
        fields: [
          { key: 'name', label: 'Nombre' },
          { key: 'element', label: 'Elemento' },
          { key: 'strength', label: 'Fuerza' },
          { key: 'cunning', label: 'Astuzia' },
          { key: 'endurance', label: 'Dureza' },
          { key: 'total', label: 'Total' },
          { key: 'type', label: 'Rareza' },
          { key: 'actions', },
        ],
        breed_dragons: []
      }
    },
    mounted(){
      if(localStorage.getItem('dragons')){
        this.dragons = JSON.parse(localStorage.getItem('dragons'))
      } else {
        this.dragons = []
        localStorage.setItem('dragons', JSON.stringify(this.dragons))
      }
    },
    methods:{
      addDragon(){
        if(localStorage.getItem('dragons')){
          this.dragons = JSON.parse(localStorage.getItem('dragons'))
        } else {
          this.dragons = []
          localStorage.setItem('dragons', JSON.stringify(this.dragons))
        }
        if(this.dragon.strength>0 && this.dragon.cunning > 0 && this.dragon.endurance > 0){
          this.dragons = JSON.parse(localStorage.getItem('dragons'))
          this.dragon.total = parseInt(this.dragon.strength) + parseInt(this.dragon.cunning) + parseInt(this.dragon.endurance)
          this.dragon.type = this.getType(this.dragon)
          this.dragons.push(this.dragon)
          this.clearValues()
          localStorage.setItem('dragons', JSON.stringify(this.dragons))
        }
      },
      clearValues(){
        this.dragon = {
          name: '',
          element: null,
          strength: 0,
          cunning: 0,
          endurance: 0,
          total: 0,
        }
      },
      getType(dragon){
        if(dragon.total <= 49){
          return 1
        } else if(dragon.total <= 89){
          return 2
        } else{
          return 3
        }
      },
      breedDragons(){
        this.breed_dragons = JSON.parse(localStorage.getItem('dragons'))
        let i = 0
        while(this.breed_dragons[this.breed_dragons.length-1].total < 90){
          this.breed_dragons.push(this.calculateStats(this.breed_dragons[i], this.breed_dragons[i+1], i))
          this.breed_dragons.push(this.calculateStats(this.breed_dragons[i], this.breed_dragons[i+1], i))
          i += 2
        }
      }, 
      calculateStats(first, second, i){
        let new_dragon = {
          name: 'breed'+i,
          element: first.element,
          strength: Math.round((this.avg(first.strength, second.strength)*0.2) + this.avg(first.strength, second.strength)),
          cunning: Math.round((this.avg(first.cunning, second.cunning)*0.2) + this.avg(first.cunning, second.cunning)),
          endurance: Math.round((this.avg(first.endurance, second.endurance)*0.2) + this.avg(first.endurance, second.endurance)),
        }
        new_dragon.total = new_dragon.strength + new_dragon.cunning + new_dragon.endurance
        new_dragon.type = this.getType(new_dragon)
        return new_dragon
      },
      avg(value1, value2){
        return ((parseInt(value1) + parseInt(value2)) / 2)
      },
      clear(){
        localStorage.removeItem('dragons')
        this.dragons = []
        this.breed_dragons = []
      }
      
    }
};
</script>
<style>
.Fire{
  padding: 5px;
  background: #d7322d !important;
  color: white;
}
.Water{
  padding: 5px;
  background: #5edbe7 !important;
  color: white;
}
.Earth{
  padding: 5px;
  background: #7b724c !important;
  color: white;
}
.Wind{
  padding: 5px;
  background: #f87fd5 !important;
  color: black;
}
.Storm{
  padding: 5px;
  background: #f9f008 !important;
  color: black;
}
.Ice{
  padding: 5px;
  background: #bbc0f1 !important;
  color: black;
}
.Plant{
  padding: 5px;
  background: #33c347 !important;
  color: black;
}
.Poison{
  padding: 5px;
  background: #420554 !important;
  color: white;
}

</style>