<template>
  <v-app>
    <v-container class="d-flex justify-center" style="min-height: 100vh;">
      <v-card class="pa-5" max-width="800" outlined>
        <!-- Title -->
        <v-card-title class="text-h4 text-center">Dompetku v2 – Expense & Income Money Management</v-card-title>
        <v-divider></v-divider>

        <!-- Form for new entry -->
        <v-form @submit.prevent="addEntry" class="mt-4">
          <v-text-field v-model="newEntry.description" label="Keterangan" outlined required></v-text-field>
          <v-text-field v-model.number="newEntry.amount" label="Jumlah Uang" type="number" outlined required></v-text-field>
          <v-select
            v-model="newEntry.category"
            :items="categories"
            label="Kategori"
            outlined
            required
          ></v-select>
          <!-- Centered Submit button -->
          <div class="d-flex justify-center mt-3">
            <v-btn type="submit" color="primary">
              Submit
            </v-btn>
          </div>
        </v-form>

        <!-- Summary Section -->
        <v-card-subtitle class="mt-5">
          <strong>Summary:</strong><br>
          Total Entries: {{ entries.length }}<br>
          Total Income: Rp {{ totalIncome }}<br>
          Total Expense: Rp {{ totalExpense }}<br>
          Balance: Rp {{ balance }}
        </v-card-subtitle>

        <!-- Rekap Keuangan header centered -->
        <v-list class="mt-5">
          <v-subheader class="text-center">Rekap Keuangan</v-subheader>
          <v-alert v-if="entries.length === 0" type="info">Belum ada data.</v-alert>
          <v-list-item
            v-for="(entry, index) in entries"
            :key="index"
            class="mb-2"
          >
            <v-list-item-content>
              <v-list-item-title>{{ entry.description }} - Rp {{ entry.amount }} ({{ entry.category }})</v-list-item-title>
            </v-list-item-content>
            <v-list-item-action>
              <v-btn icon color="red" @click="removeEntry(index)">
                <v-icon>mdi-delete</v-icon>
              </v-btn>
            </v-list-item-action>
          </v-list-item>
        </v-list>

        <!-- Clear all entries button centered -->
        <div class="d-flex justify-center mt-5">
          <v-btn color="error" @click="confirmClearAll">
            Hapus Semua Data
          </v-btn>
        </div>
      </v-card>
    </v-container>
  </v-app>
</template>

<script>
import { ref, computed, onMounted } from 'vue';
import { loadEntries, saveEntries, clearEntries } from '../stores/store.js';

export default {
  data() {
    return {
      newEntry: {
        description: "",
        amount: null,
        category: "",
      },
      categories: ["Food", "Transport", "Shopping", "Other"],
      entries: loadEntries()
    };
  },
  computed: {
    totalIncome() {
      return this.entries
        .filter(entry => entry.amount > 0)
        .reduce((sum, entry) => sum + entry.amount, 0);
    },
    totalExpense() {
      return this.entries
        .filter(entry => entry.amount < 0)
        .reduce((sum, entry) => sum + Math.abs(entry.amount), 0);
    },
    balance() {
      return this.totalIncome - this.totalExpense;
    }
  },
  methods: {
    addEntry() {
      if (this.newEntry.description && this.newEntry.amount && this.newEntry.category) {
        this.entries.push({ ...this.newEntry });
        saveEntries(this.entries);
        this.newEntry = { description: "", amount: null, category: "" };
      } else {
        alert("Harap isi keterangan, jumlah uang, dan kategori.");
      }
    },
    removeEntry(index) {
      this.entries.splice(index, 1);
      saveEntries(this.entries);
    },
    confirmClearAll() {
      if (confirm("Apakah Anda yakin ingin menghapus semua data?")) {
        this.clearAllEntries();
      }
    },
    clearAllEntries() {
      this.entries = [];
      clearEntries();
    }
  },
  onMounted() {
    this.entries = loadEntries();
  }
};
</script>

<style scoped>
@import "../../node_modules/vuetify/dist/vuetify.min.css";
</style>
