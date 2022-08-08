<template>
  <v-container >
    <v-row class="text-center">
      <v-col class="my-4" cols="12">
        <v-card outlined>
          <v-container fluid>
            <v-row align="center">
              <v-col>
                <CustomSlider :min="1" :max="40" text="Individual length" @valueChange="individualLength = $event"/>
                <CustomSlider :min="10" :max="100" :steps="5" text="Population size" @valueChange="populationSize = $event"/>
              </v-col>
              <v-col class="d-flex flex-column align-stretch">
                <v-row>
                  <v-select v-model="selectionMethod" :items="selectionMethods"
                  label="Selection">
                  </v-select>
                </v-row>
                <v-row v-show="selectionMethod == 'Best'">
                  <CustomSlider :min="0" :max="100" :steps="5" text="Keep top X %" @valueChange="selectionBestTopPercentage = $event"/>
                </v-row>
                <v-row>
                  <v-select v-model="crossOverMethod" :items="crossOverMethods"
                  label="Cross-over">
                  </v-select>
                </v-row>
                <CustomSlider :min="0" :max="15" text="Mutation rate (in %)" @valueChange="mutationRate = $event"/>
              </v-col>
              <v-col>
                <CustomSlider :min="100" :max="1000" :steps="100" text="Max iterations" @valueChange="iterations = $event"/>
              </v-col>
            </v-row>
          </v-container>
          <v-card-actions>
            <v-btn outlined @click="generatePopulation()">
              Generate
            </v-btn>
            <v-btn outlined @click="sortPopulation()">
              Sort
            </v-btn>
            <v-btn outlined @click="selectFromPopulation()" :disabled="selectionMethod == null">
              Select
            </v-btn>
            <v-btn outlined @click="mutatePopulation()">
              Mutate
            </v-btn>
            <v-btn outlined @click="crossOverPopulation()">
              Cross-over
            </v-btn>
            <v-btn outlined @click="createNewGeneration()">
              Create new
            </v-btn>
            <v-btn outlined @click="changeCurrentGeneration()">
              Pass to next
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-col>

      <v-col cols=12>
        <v-row>  
          <v-col class="mb-5">
            <h2 class="headline font-weight-bold mb-3">
              Population
            </h2>

            <v-row justify="center">
              {{ populationSize }}
            </v-row>
          </v-col>

          <v-col class="mb-5">
            <h2 class="headline font-weight-bold mb-3">
              Mutation
            </h2>

            <v-row justify="center">
              {{ mutationRate }} %
            </v-row>
          </v-col>

          <v-col class="mb-5">
            <h2 class="headline font-weight-bold mb-3">
              Max iterations
            </h2>

            <v-row justify="center">
              {{ iterations }}
            </v-row>
          </v-col>
        </v-row>
      </v-col>

      <v-col>
        <PopulationDisplay :items="population" title="Population"/>
      </v-col>
      <v-col>
        <PopulationDisplay :items="sortedPopulation" title="Sorted population"/>
      </v-col>
      <v-col>
        <PopulationDisplay :items="selectedPopulation" title="Selected population"/>
      </v-col>
      <v-col>
        <PopulationDisplay :items="mutatedPopulation" title="Mutated population"/>
      </v-col>
      <v-col>
        <PopulationDisplay :items="crossedOverPopulation" title="Crossed over population"/>
      </v-col>
      <v-col>
        <PopulationDisplay :items="newGeneration" title="New generation"/>
      </v-col>

    </v-row>

    <v-row>
      <apexchart width="500" type="bar" :options="computedOptions" :series="computedSeries"></apexchart>
    </v-row>
  </v-container>
</template>

<script>
  import CustomSlider from "./settings/CustomSlider.vue"
  import PopulationDisplay from "./visuals/PopulationDisplay.vue"

  export default {
    name: 'MainThread',

    components: {
      CustomSlider,
      PopulationDisplay,
    },

    computed: {
      allSettings: {
        get: function() {
          return "" + (this.selectionMethod || "") + this.selectionBestTopPercentage + (this.crossOverMethod || "") + 
            this.populationSize + this.individualLength + this.mutationRate + this.iterations;
        }
      },
      computedOptions: {
        get: function() {
          return {
            yaxis: { min: 0, max: 1 },
            xaxis: {
              categories: Array(this.populationSize).map((v,i)=>i+1)
            }
          }
        }
      },
      computedSeries: {
        get: function() {
          return [{
            name: 'Fitness evolution',
            data: this.population.map(v => parseFloat(this.computeFitness(v)/this.individualLength).toFixed(2))
          }]
        }
      },
      averageFitness: {
        get: function() {
          return this.population.map(individual => this.computeFitness(individual)/this.individualLength).reduce((acc, val) => acc + val, 0)/this.populationSize;
        },
      },
    },

    watch: {
      allSettings: {
        handler() {
          this.generatePopulation()
        }
      },
      population: {
        handler() {
          this.sortPopulation()
        }
      },
      sortedPopulation: {
        handler() {
          this.selectFromPopulation()
        }
      },
      selectedPopulation: {
        handler() {
          this.mutatePopulation()
        }
      },
      mutatedPopulation: {
        handler() {
          this.crossOverPopulation()
        }
      },
    },

    data: () => ({
      // Settings
      selectionMethod: null,
      selectionMethods: ['Best', 'Tournament'],
      selectionBestTopPercentage: 0,
      crossOverMethod: null,
      crossOverMethods: ['1 out of 2', 'Split'],
      populationSize: 0,
      individualLength: 0,
      mutationRate: 0,
      iterations: 0,
      // Items
      population: [],
      sortedPopulation: [],
      selectedPopulation: [],
      mutatedPopulation: [],
      crossedOverPopulation: [],
      newGeneration: [],
      // Chart
      chartOptions: {
        xaxis: {
          categories: [1, 2, 3, 4, 5, 6, 7, 8]
        }
      },
      series: [{
        name: 'Fitness evolution',
        data: [30, 40, 35, 50, 49, 60, 70, 91]
      }]
    }),

    methods: {
      getRandomBit() {
        return Math.random() > 0.5 ? 1 : 0;
      },
      getRandomInt() {
        return parseInt(100 * Math.random());
      },
      generateIndividual() {
        let s = "";
        for (let i = 0; i < this.individualLength; i++) {
          s += this.getRandomBit();
        }
        return s;
      },
      generateMultipleIndividuals(number) {
        let population = [];
        for (let i = 0; i < number; i++) {
          population.push(this.generateIndividual())
        }
        return population; 
      },
      generatePopulation() {
        this.population = this.generateMultipleIndividuals(this.populationSize);
      },
      computeFitness(individual = "") {
        return individual.split("").map(i => parseInt(i)).reduce((acc, val) => acc + val, 0);
      },
      sortPopulation() {
        this.sortedPopulation = Array.from(this.population);
        this.sortedPopulation.sort((a,b) => this.computeFitness(b) - this.computeFitness(a))
      },
      selectFromPopulation() {
        this.selectedPopulation = this.applySelectionMethod(this.sortedPopulation, this.selectionMethod);
      },
      applySelectionMethod(population = [], method) {
        if (method == "Best") {
          let elementsToKeep = parseInt(population.length * this.selectionBestTopPercentage / 100);
          return population.slice(0,elementsToKeep);
        }
        return [];
      },
      mutateGene(gene) {
        return "" + ((parseInt(gene) + 1) % 2)
      },
      mutateIndividual(individual = "") {
        for (let i = 0; i < individual.length; i++) {
          if (this.getRandomInt() < this.mutationRate) {
            let tmp = individual.split("")
            tmp[i] = this.mutateGene(individual[i]);
            individual = tmp.join("");
          }
        }
        return individual;
      },
      mutatePopulation() {
        this.mutatedPopulation = this.selectedPopulation.map(i => this.mutateIndividual(i));
      },
      applyCrossOverMethod(a, b) {
        let newBorn = "";
        if (this.crossOverMethod == "1 out of 2") {
          for (let i = 0; i < this.individualLength; i++) {
            newBorn += (this.getRandomBit() == 0) ? a[i] : b[i]
          }
        } else if (this.crossOverMethod == "Split") {
          newBorn = (this.getRandomBit() == 0)
            ? a.slice(0,this.individualLength/2) + b.slice(this.individualLength/2, this.individualLength)
            : b.slice(0,this.individualLength/2) + a.slice(this.individualLength/2, this.individualLength); 
        }
        return newBorn;
      },
      crossOverIndividuals(a, b) {
        return this.applyCrossOverMethod(a, b);
      },
      crossOverPopulation() {
        this.crossedOverPopulation = Array.from(this.mutatedPopulation);
        if (this.crossedOverPopulation.length > 2) {
          let alpha = this.crossedOverPopulation[0];
          let beta = this.crossedOverPopulation[1];
          let gamma = this.crossedOverPopulation[2];
          this.crossedOverPopulation.push(this.crossOverIndividuals(alpha, beta));
          this.crossedOverPopulation.push(this.crossOverIndividuals(alpha, gamma));
          this.crossedOverPopulation.push(this.crossOverIndividuals(beta, gamma));
        }
      },
      createNewGeneration() {
        this.newGeneration = [].concat(
          this.crossedOverPopulation,
          this.generateMultipleIndividuals(this.populationSize - this.crossedOverPopulation.length)
        )
      },
      changeCurrentGeneration() {
        this.population = Array.from(this.newGeneration);
      },
    },
  }
</script>
