<template>
  <div>
    <div class="main-container">
      <!-- Filtros -->
      <aside class="filter-container">

        <div class="filter-container-inner">
          <h4 class="filter-subtitle">Marcas</h4>
          <Splide
            :options="{
              type: 'loop', // Define o carrossel como loop
              perPage: 6, // Quantos slides serão exibidos ao mesmo tempo
              gap: '0rem', // Espaçamento entre os slides
              pagination: false, // Remove a paginação
              arrows: false, // Exibe as setas de navegação
              fixedWidth: '50px', // Largura fixa dos slides
              focus: 'center', // Centraliza o slide ativo
              breakpoints: {
                768: { perPage: 3, fixedWidth: '60px' }, // Ajusta para telas menores
                480: { perPage: 2, fixedWidth: '50px' }, // Ajusta para telas muito pequenas
              }
            }"
            class="brand-splide"
          >
            <SplideSlide v-for="marca in uniqueBrands" :key="marca">
              <div class="brand-filter-splide" @click="filterByBrand(marca)">
                <img :src="getBrandImage(marca)" :alt="marca" class="brand-logo" />
              </div>
            </SplideSlide>
          </Splide>

          <h4 class="filter-subtitle">Selecione a Marca</h4>
          <select v-model="selectedMarca" @change="applyFilters" class="filter-select">
            <option value="" disabled selected>Selecione uma marca</option>
            <option v-for="marca in uniqueBrands" :key="marca" :value="marca">{{ marca }}</option>
          </select>

          <h4 class="filter-subtitle">Faixa de Preço</h4>
          <div class="filter-range-group">
            <input
              type="number"
              v-model="priceFilter.min"
              placeholder="Mínimo"
              @input="applyFilters"
              class="filter-input"
            />
            <input
              type="number"
              v-model="priceFilter.max"
              placeholder="Máximo"
              @input="applyFilters"
              class="filter-input"
            />
          </div>

          <h4 class="filter-subtitle">Quilometragem</h4>
          <select v-model="kmFilter.maxRange" @change="applyFilters" class="filter-select">
            <option value="">Todas</option>
            <option value="0">0km</option>
            <option value="0-10">0-10mil km</option>
            <option value="10-50">10-50mil km</option>
            <option value="50-100">50-100mil km</option>
            <option value="100-200">100-200mil km</option>
            <option value="200+">200+ mil km</option>
          </select>

          <h4 class="filter-subtitle">Condição</h4>
          <select v-model="conditionFilter" @change="applyFilters" class="filter-select">
            <option value="">Todas</option>
            <option value="Novo">Novo</option>
            <option value="Usado">Usado</option>
          </select>

          <h4 class="filter-subtitle">Portas</h4>
          <select v-model="portasFilter" @change="applyFilters" class="filter-select">
            <option value="">Todas</option>
            <option value="2">2 Portas</option>
            <option value="4">4 Portas</option>
          </select>

          <h4 class="filter-subtitle">Tração</h4>
          <select v-model="tractionFilter" @change="applyFilters" class="filter-select">
            <option value="">Todas</option>
            <option value="4x2">4x2</option>
            <option value="4x4">4x4</option>
            <option value="AWD">AWD</option>
          </select>

          <h4 class="filter-subtitle">Transmissão</h4>
          <select v-model="transmissionFilter" @change="applyFilters" class="filter-select">
            <option value="">Todas</option>
            <option value="Manual">Manual</option>
            <option value="Automática">Automática</option>
          </select>

          <h4 class="filter-subtitle">Carroceria</h4>
          <select v-model="carBodyFilter" @change="applyFilters" class="filter-select">
            <option value="">Todas</option>
            <option value="Hatch">Hatch</option>
            <option value="Sedan">Sedan</option>
            <option value="SUV">SUV</option>
          </select>

          <h4 class="filter-subtitle">Opcionais</h4>
          <div v-for="opcional in opcionaisDisponiveis" :key="opcional" class="filter-checkbox-group">
            <label>
              <input
                type="checkbox"
                :value="opcional"
                v-model="filters.opcionais"
                @change="applyFilters"
                class="filter-checkbox"
              />
              {{ opcional }}
            </label>
          </div>
        </div>
        <button class="filter-btn-clear" @click="clearFilters">Limpar</button>
      </aside>

      <!-- Grade de Veículos -->
      <div class="vehicle-grid">
        <div v-for="vehicle in filteredVehicles" :key="vehicle.id" class="vehicle-card">
          <div class="vehicle-image">
            <Splide :options="{ type: 'loop', autoplay: true, interval: 3000 }">
              <SplideSlide v-for="foto in vehicle.fotos" :key="foto">
                <img :src="`http://localhost:5000/uploads/${foto}`" alt="Vehicle photo" />
              </SplideSlide>
            </Splide>
            <button
              class="favorite-btn"
              @click="toggleFavorite(vehicle)"
              :class="{ active: isFavorite(vehicle.id) }"
            >
              ❤
            </button>
          </div>
          <div class="vehicle-info">
            <h3>{{ vehicle.marca || "Não informado" }}</h3>
            <h4>{{ vehicle.modelo || "Não informado" }}</h4>
            <p class="price">R$ {{ Math.floor(vehicle.preco).toLocaleString('pt-BR') }}</p>
            <p>
              <strong>Ano:</strong> {{ vehicle.ano || "Não informado" }} |
              <strong>Quilometragem:</strong>
              {{ vehicle.quilometragem ? vehicle.quilometragem.toLocaleString() + " Km" : "Não informado" }}
            </p>
            <p><strong>Transmissão:</strong> {{ vehicle.transmissao || "Não informado" }}</p>
            <p><strong>Carroceria:</strong> {{ vehicle.carroceria || "Não informado" }}</p>
            <p><strong>Cor:</strong> {{ vehicle.cor || "Não informado" }}</p>
            <button class="btn" @click="viewDetails(vehicle.id)">Ver Detalhes</button>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>
<script>
import "@/assets/VehicleStyles.css";
import axios from "axios";
import "@splidejs/vue-splide/css";
import { Splide, SplideSlide } from "@splidejs/vue-splide";

export default {
  name: "ViewVehicles",
  components: {
    Splide,
    SplideSlide,
  },
  data() {
    return {
      vehicles: [],
      filteredVehicles: [],
      favorites: JSON.parse(localStorage.getItem("favorites")) || [],
      searchQuery: "",
      uniqueBrands: [],
      opcionaisDisponiveis: ["Banco de couro", "Teto solar", "Câmera de ré", "Ar-condicionado"],
      filters: {
        opcionais: [],
      },
      priceFilter: {
        min: null,
        max: null,
      },
      kmFilter: {
        maxRange: "", // Alteração aqui para suportar o seletor de quilometragem
      },
      conditionFilter: "",
      portasFilter: "",
      tractionFilter: "",
      transmissionFilter: "",
      carBodyFilter: "",
      colorFilter: "",
    };
  },
  methods: {
    async fetchVehicles() {
      try {
        const response = await axios.get("http://localhost:5000/api/vehicles");
        this.vehicles = response.data.map((vehicle) => ({
          ...vehicle,
          quilometragem: vehicle.quilometragem || 0,
          preco: vehicle.preco || 0,
          opcionais: vehicle.opcionais || [],
          fotos: vehicle.fotos || [],
        }));
        this.uniqueBrands = [...new Set(this.vehicles.map((v) => v.marca))];
        this.applyInitialFilter();
      } catch (err) {
        alert("Erro ao buscar veículos.");
      }
    },
    matchKilometragem(quilometragem) {
      if (this.kmFilter.maxRange === "0") {
        return quilometragem === 0;
      } else if (this.kmFilter.maxRange === "0-10") {
        return quilometragem > 0 && quilometragem <= 10000;
      } else if (this.kmFilter.maxRange === "10-50") {
        return quilometragem > 10000 && quilometragem <= 50000;
      } else if (this.kmFilter.maxRange === "50-100") {
        return quilometragem > 50000 && quilometragem <= 100000;
      } else if (this.kmFilter.maxRange === "100-200") {
        return quilometragem > 100000 && quilometragem <= 200000;
      } else if (this.kmFilter.maxRange === "200+") {
        return quilometragem > 200000;
      }
      return true; // Retorna true para casos onde não há filtro aplicado
    },
    toggleFavorite(vehicle) {
      const index = this.favorites.findIndex((fav) => fav.id === vehicle.id);
      if (index > -1) {
        this.favorites.splice(index, 1);
      } else {
        this.favorites.push(vehicle);
      }
      localStorage.setItem("favorites", JSON.stringify(this.favorites));
    },
    isFavorite(id) {
      return this.favorites.some((fav) => fav.id === id);
    },
    filterByBrand(brand) {
      this.filteredVehicles = this.vehicles.filter((vehicle) => vehicle.marca === brand);
    },
    applyFilters() {
      this.filteredVehicles = this.vehicles.filter((vehicle) => {
        const searchMatch = `${vehicle.marca} ${vehicle.modelo}`.toLowerCase().includes(this.searchQuery.toLowerCase());
        const priceMatch =
          (!this.priceFilter.min || vehicle.preco >= this.priceFilter.min) &&
          (!this.priceFilter.max || vehicle.preco <= this.priceFilter.max);
        const kmMatch = !this.kmFilter.maxRange || this.matchKilometragem(vehicle.quilometragem);
        const conditionMatch = !this.conditionFilter || vehicle.condicao === this.conditionFilter;
        const portasMatch = !this.portasFilter || vehicle.portas === parseInt(this.portasFilter);
        const tractionMatch = !this.tractionFilter || vehicle.driveType === this.tractionFilter;
        const transmissionMatch = !this.transmissionFilter || vehicle.transmissao === this.transmissionFilter;
        const carBodyMatch = !this.carBodyFilter || vehicle.carroceria === this.carBodyFilter;
        const colorMatch = !this.colorFilter || vehicle.cor.toLowerCase().includes(this.colorFilter.toLowerCase());
        const opcionaisMatch =
          this.filters.opcionais.length === 0 ||
          this.filters.opcionais.every((opcional) => vehicle.opcionais.includes(opcional));
        return (
          searchMatch &&
          priceMatch &&
          kmMatch &&
          conditionMatch &&
          portasMatch &&
          tractionMatch &&
          transmissionMatch &&
          carBodyMatch &&
          colorMatch &&
          opcionaisMatch
        );
      });
    },

    applyInitialFilter() {
      const brand = this.$route.query.brand;
      if (brand) {
        this.filterByBrand(brand);
      } else {
        this.filteredVehicles = [...this.vehicles];
      }
    },
    viewDetails(id) {
      this.$router.push(`/details/${id}`);
    },
    getBrandImage(brand) {
      const normalizedBrand = brand.toLowerCase().replace(/\s+/g, "-");
      try {
        return require(`@/assets/images/logos/${normalizedBrand}.webp`);
      } catch {
        return "/src/assets/images/logos/default.png"; // Caminho para uma imagem padrão
      }
    },
    clearFilters() {
      this.selectedMarca = "";
      this.priceFilter.min = null;
      this.priceFilter.max = null;
      this.kmFilter.max = null;
      this.conditionFilter = "";
      this.portasFilter = "";
      this.tractionFilter = "";
      this.transmissionFilter = "";
      this.carBodyFilter = "";
      this.colorFilter = "";
      this.filters.opcionais = [];
      this.applyFilters();
    },
  },
  watch: {
    "$route.query.brand": function (newBrand) {
      if (newBrand) {
        this.filterByBrand(newBrand);
      } else {
        this.filteredVehicles = [...this.vehicles];
      }
    },
  },
  created() {
    this.fetchVehicles();
  },
};
</script>