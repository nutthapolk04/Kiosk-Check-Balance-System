<script setup lang="ts">
import { ref, computed } from 'vue';
import { useRouter } from 'vue-router';
import { useKioskStore } from '../stores/kioskStore';
import { ChevronLeft, ArrowUpCircle, ArrowDownCircle, ReceiptText } from 'lucide-vue-next';
import ReceiptPreview from '../components/ReceiptPreview.vue';
import type { Transaction } from '../api/mockApi';

const router = useRouter();
const store = useKioskStore();
const currT = computed(() => t[store.language as 'EN' | 'TH']);

const selectedTransaction = ref<Transaction | null>(null);
const showReceipt = ref(false);

const t = {
  EN: {
    title: 'Transactions',
    back: 'Back',
    topup: 'Top-up',
    purchase: 'Purchase',
    noData: 'No transactions found.',
    qty: 'x'
  },
  TH: {
    title: 'ประวัติรายการ',
    back: 'ย้อนกลับ',
    topup: 'เติมเงิน',
    purchase: 'ซื้อสินค้า',
    noData: 'ไม่พบรายการ',
    qty: 'x'
  }
};

const goBack = () => {
  router.push('/balance');
};

const openReceipt = (tx: Transaction) => {
  selectedTransaction.value = tx;
  showReceipt.value = true;
};

const closeReceipt = () => {
  showReceipt.value = false;
};

const formatCurrency = (val: number) => {
  return new Intl.NumberFormat(store.language === 'TH' ? 'th-TH' : 'en-US', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2
  }).format(val);
};
</script>

<template>
  <div class="kiosk-container history-view">
    <div class="header-section">
      <button class="back-btn" @click="goBack">
        <ChevronLeft :size="32" />
        <span>{{ currT.back }}</span>
      </button>
      <h2>{{ currT.title }}</h2>
      <div style="width: 100px"></div>
    </div>

    <div class="list-container" v-if="store.transactions.length > 0">
      <div 
        v-for="tx in store.transactions" 
        :key="tx.id" 
        class="transaction-row card"
        @click="openReceipt(tx)"
      >
        <div class="tx-icon" :class="tx.type">
          <ArrowUpCircle v-if="tx.type === 'topup'" :size="32" />
          <ArrowDownCircle v-else :size="32" />
        </div>
        
        <div class="tx-details">
          <div class="tx-main">
            <div class="tx-left">
              <span class="tx-type">
                {{ tx.type === 'topup' ? currT.topup : currT.purchase }}
                <span v-if="tx.type === 'purchase'" class="tx-shop"> - {{ tx.machine }}</span>
              </span>
              <!-- Purchase items list -->
              <ul v-if="tx.items && tx.items.length" class="tx-items">
                <li v-for="(item, i) in tx.items" :key="i">
                  {{ item.name }}
                  <span v-if="item.qty > 1"> {{ currT.qty }}{{ item.qty }}</span>
                  <span class="item-price">฿{{ formatCurrency(item.price * item.qty) }}</span>
                </li>
              </ul>
            </div>
            <span class="tx-amount" :class="tx.type">
              {{ tx.type === 'topup' ? '+' : '-' }} {{ formatCurrency(tx.amount) }}
            </span>
          </div>
          <div class="tx-meta">
            <span>{{ tx.date }} {{ tx.time }}</span>
            <span v-if="tx.type === 'topup'">• {{ tx.machine }}</span>
          </div>
        </div>
        
        <div class="tx-action">
          <ReceiptText :size="24" />
        </div>
      </div>
    </div>

    <div v-else class="empty-state">
      <p>{{ currT.noData }}</p>
    </div>

    <ReceiptPreview 
      v-if="showReceipt && selectedTransaction" 
      :transaction="selectedTransaction" 
      @close="closeReceipt" 
    />
  </div>
</template>

<style scoped>
.history-view {
  padding: 2rem;
  justify-content: flex-start;
}

.header-section {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 2rem;
}

.back-btn {
  background: none;
  border: none;
  color: var(--text-color);
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 1.25rem;
  font-weight: 700;
  cursor: pointer;
}

.list-container {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 1rem;
  overflow-y: auto;
  padding-bottom: 2rem;
}

.transaction-row {
  display: flex;
  align-items: center;
  padding: 1.5rem;
  gap: 1.5rem;
  cursor: pointer;
  transition: transform 0.1s;
}

.transaction-row:active {
  transform: scale(0.98);
  background-color: var(--bg-color);
}

.tx-icon.topup { color: #10b981; }
.tx-icon.purchase { color: #ef4444; }

.tx-details {
  flex: 1;
}

.tx-main {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.25rem;
}

.tx-type {
  font-weight: 700;
  font-size: 1.25rem;
}

.tx-amount {
  font-weight: 800;
  font-size: 1.5rem;
}

.tx-amount.topup { color: #10b981; }
.tx-amount.purchase { color: #ef4444; }

.tx-meta {
  color: var(--text-muted);
  font-size: 1rem;
}

.tx-action {
  color: var(--text-muted);
  opacity: 0.5;
}

.tx-left {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  flex: 1;
  min-width: 0;
}

.tx-shop {
  font-weight: 600;
  color: var(--text-muted);
  font-size: 0.9em;
}

.tx-items {
  list-style: none;
  padding: 0;
  margin: 0.25rem 0 0 0;
  display: flex;
  flex-direction: column;
  gap: 0.15rem;
}

.tx-items li {
  font-size: 0.85rem;
  color: var(--text-muted);
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.tx-items li::before {
  content: '•';
  color: #cbd5e1;
  font-weight: 700;
}

.item-price {
  margin-left: auto;
  font-weight: 600;
  font-size: 0.8rem;
  color: #64748b;
}

.empty-state {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--text-muted);
  font-style: italic;
}
</style>
