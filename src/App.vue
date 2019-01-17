<template>
  <main>
    <b-navbar toggleable="sm" type="dark" variant="info">
      <b-container>
        <b-navbar-brand>Посчитай </b-navbar-brand>
        <b-navbar-toggle target="nav_collapse"></b-navbar-toggle>
        <b-collapse is-nav id="nav_collapse">
          <b-navbar-nav>
            <b-nav-item>налоги с зарплаты </b-nav-item>
          </b-navbar-nav>
          <b-navbar-nav class="ml-auto">
            <b-nav-item href="https://ecomarine.kz">EcoMarine</b-nav-item>
          </b-navbar-nav>
        </b-collapse>
      </b-container>
    </b-navbar>
    <b-container>
      <b-row>
        <b-col md="8">
          <h2>Введите вашу зарплату в месяц</h2>
          <!-- {{ rate }} -->
          <b-row class="row">
            <b-col sm="5" cols="7">
              <b-input-group size="lg">
                <b-form-input
                  v-model="inputSalary"
                  v-on:change="calculate"
                  type="number"
                ></b-form-input>
                <div class="input-group-append">
                  <span class="input-group-text">₸</span>
                </div>
              </b-input-group>
            </b-col>
            <b-col sm="7" cols="5">
              <b-input-group size="lg">
                <span class="input-group-text">
                  <b-form-checkbox
                    size="lg"
                    plain
                    v-model="check"
                    value="gross"
                    unchecked-value="net"
                    v-on:change="calculate"
                    >На руки</b-form-checkbox
                  >
                </span>
              </b-input-group>
            </b-col>
          </b-row>
        </b-col>
      </b-row>
      <div class="row">
        <b-col md="6">
          <h2>Расчеты</h2>
          <b-list-group>
            <b-list-group-item
              class="d-flex justify-content-between align-items-center"
            >
              Оклад
              <b-badge variant="primary">
                + {{ formatResult(result.netSalary) }}
              </b-badge>
            </b-list-group-item>
            <b-list-group-item
              class="d-flex justify-content-between align-items-center"
            >
              <abbr v-b-tooltip.hover title="Обязательный пенсионный взнос">
                ОПВ
              </abbr>
              <b-badge variant="danger">
                - {{ formatResult(result.pension) }}
              </b-badge>
            </b-list-group-item>
            <b-list-group-item
              class="d-flex justify-content-between align-items-center"
            >
              <abbr v-b-tooltip.hover title="Индивидуальный подоходный налог">
                ИПН
              </abbr>
              <b-badge variant="danger">
                - {{ formatResult(result.tax) }}
              </b-badge>
            </b-list-group-item>
            <b-list-group-item
              class="d-flex justify-content-between align-items-center"
            >
              <span v-b-tooltip.hover title="Доход на руки">
                Заработная плата
              </span>
              <b-badge variant="success">
                = {{ formatResult(result.salary) }}
              </b-badge>
            </b-list-group-item>
            <b-list-group-item
              class="d-flex justify-content-between align-items-center"
            >
              Заработная плата за год
              <b-badge variant="info">
                {{ formatResult(result.salary * 12) }} <br />
                <small>{{ toUSD(result.salary * 12) }}</small> |
                <small>{{ toEUR(result.salary * 12) }}</small>
              </b-badge>
            </b-list-group-item>
            <b-list-group-item
              class="d-flex justify-content-between align-items-center"
            >
              Оклад за год
              <b-badge variant="info">
                {{ formatResult(result.netSalary * 12) }} <br />
                <small>{{ toUSD(result.netSalary * 12) }}</small> |
                <small>{{ toEUR(result.netSalary * 12) }}</small>
              </b-badge>
            </b-list-group-item>
          </b-list-group>
          <!--
            <script src="//yastatic.net/es5-shims/0.0.2/es5-shims.min.js"></script>
            <script src="//yastatic.net/share2/share.js"></script>
            <div class="ya-share2" data-services="vkontakte,facebook,twitter,linkedin,whatsapp,telegram" data-counter=""></div>
          -->
        </b-col>

        <b-col md="4" class="text-light">
          <share-facebook title_social="Facebook" has_counter></share-facebook>
          <share-vkontakte title_social="VK.com" has_counter></share-vkontakte>
          <share-twitter title_social="Twitter"></share-twitter>
          <share-telegram title_social="Telegram"></share-telegram>
          <share-whatsapp title_social="WhatsApp"></share-whatsapp>
        </b-col>
      </div>
    </b-container>
  </main>
</template>

<script>
import ShareFacebook from "vue-goodshare/src/providers/Facebook";
import ShareTwitter from "vue-goodshare/src/providers/Twitter";
import ShareVkontakte from "vue-goodshare/src/providers/Vkontakte";
import ShareTelegram from "vue-goodshare/src/providers/Telegram";
import ShareWhatsapp from "vue-goodshare/src/providers/WhatsApp";
export default {
  name: "app",
  components: {
    ShareFacebook,
    ShareTwitter,
    ShareVkontakte,
    ShareTelegram,
    ShareWhatsapp
  },
  data() {
    return {
      currentYear: 2018,
      minimalSalary: 28284,
      inputSalary: 28284,
      check: "gross",
      result: {
        netSalary: 0,
        pension: 0,
        tax: 0,
        salary: 0
      },
      rate: {
        usd: 321,
        eur: 396
      }
    };
  },
  beforeMount() {
    switch (this.currentYear) {
      case 2018:
        this.minimalSalary = 28284;
        break;
      default:
        this.currentYear = 2018;
        this.minimalSalary = 28284;
    }
    const todaysDate = new Date();
    const storedDate = new Date(localStorage.getItem("today"));
    if (
      localStorage.getItem("rate") &&
      todaysDate.setHours(0, 0, 0, 0) === storedDate.setHours(0, 0, 0, 0)
    ) {
      this.rate = JSON.parse(localStorage.getItem("rate"));
    } else {
      const apiUrl =
        "https://openexchangerates.org/api/latest.json?app_id=0a7936c4fdc04243a2a994352484ae28";
      fetch(apiUrl)
        .then(response => response.json())
        .then(data => {
          this.rate.usd = Math.round(data.rates.KZT * 100) / 100;
          this.rate.eur = Math.round(this.rate.usd / data.rates.EUR);
          localStorage.setItem("today", todaysDate);
          localStorage.setItem("rate", JSON.stringify(this.rate));
          this.calculate();
        });
    }
    this.calculate();
  },
  methods: {
    calculate() {
      if (this.check === "net") {
        this.check = "gross";
        this.result.netSalary =
          (this.inputSalary - this.minimalSalary * 0.1) / 0.81;
        this.result.pension =
          this.result.netSalary * 0.1 < this.minimalSalary * 75
            ? this.result.netSalary * 0.1
            : this.minimalSalary * 75;
        this.result.tax =
          this.result.netSalary === this.minimalSalary
            ? 0
            : (this.result.netSalary -
                this.result.pension -
                this.minimalSalary) *
              0.1;
        this.result.tax += 0;
        this.result.salary = this.inputSalary;
      } else if (this.check === "gross") {
        this.check = "net";
        this.result.netSalary = this.inputSalary;
        this.result.pension =
          this.result.netSalary * 0.1 < this.minimalSalary * 75
            ? this.result.netSalary * 0.1
            : this.minimalSalary * 75;
        this.result.tax =
          this.result.netSalary === this.minimalSalary
            ? 0
            : (this.result.netSalary -
                this.result.pension -
                this.minimalSalary) *
              0.1;
        this.result.tax += 0;
        this.result.salary =
          this.result.netSalary - this.result.pension - this.result.tax;
      }
    },
    formatResult(value) {
      let val = (Math.abs(value) / 1).toFixed(2).replace(".", ",");
      val += " ₸";
      return val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, " ");
    },
    toUSD(salary) {
      let val = parseInt(salary, 10) / this.rate.usd;
      val = (val / 1).toFixed(0).replace(".", ",");
      val = `$${val}`;
      return val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, " ");
    },
    toEUR(salary) {
      let val = parseInt(salary, 10) / this.rate.eur;
      val = (val / 1).toFixed(0).replace(".", ",");
      val = `${val}€`;
      return val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, " ");
    }
  }
};
</script>

<style>
.input-group-text {
  padding: 0.7rem 0.75rem;
}
</style>
