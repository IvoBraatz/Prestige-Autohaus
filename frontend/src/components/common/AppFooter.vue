<template>
  <footer class="footer-section">
    <div class="footer-container">
      <div class="footer-logo" data-aos="fade-right">
        <h3>Element</h3>
        <p>Seu parceiro confiável em soluções tecnológicas e inovação.</p>
        <p>&copy; 2024 Element. All rights reserved.</p>
      </div>
      <div class="footer-contact" data-aos="fade-left">
        <h4>Fale Conosco</h4>
        <ul>
          <li>
            <i class="fas fa-phone"></i>+55 47 997486918
          </li>
          <li>
            <i class="fas fa-envelope"></i>contato@element.dev.br
          </li>
          <li>
            <i class="fas fa-map-marker-alt"></i>Timbó, SC, Brasil
          </li>
        </ul>
      </div>
    </div>
  </footer>
</template>
<script>
import AOS from "aos";
import "aos/dist/aos.css"; // Importa os estilos do AOS
import "@/assets/AppFooter.css";

export default {
  name: "AppFooter",
  mounted() {
    AOS.init({
      duration: 1000, // Duração da animação em milissegundos
      once: true, // Se true, anima apenas na primeira vez que o elemento aparece
    });
  },
};
</script>

