<template>
  <teleport to="body">
    <div v-if="modelValue" class="bmc-overlay" @click.self="$emit('skip')">
      <div class="bmc-modal">
        <div class="bmc-icon">☕</div>
        <h2 class="bmc-title">Sviđa vam se alat?</h2>
        <p class="bmc-text">Ako vam generator uplatnica štedi vrijeme, razmislite o podršci razvoju.</p>
        <div class="bmc-actions">
          <a
            class="bmc-coffee-btn"
            href="https://buymeacoffee.com/sparkschess"
            target="_blank"
            rel="noopener"
            @click="$emit('skip')"
          >☕ Podržite razvoj</a>
          <button class="bmc-skip-btn" @click="$emit('skip')">Nastavi generiranje</button>
        </div>
      </div>
    </div>
  </teleport>
</template>

<script>
export default {
  name: 'BmcModal',
  props: {
    modelValue: { type: Boolean, default: false },
  },
  emits: ['skip'],
};
</script>

<style>
.bmc-overlay {
  position: fixed;
  inset: 0;
  background: rgba(10, 20, 60, 0.55);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
}

.bmc-modal {
  background: white;
  border-radius: 20px;
  padding: 2.25rem 2.5rem;
  max-width: 390px;
  width: 90%;
  text-align: center;
  box-shadow: 0 20px 60px rgba(10, 20, 60, 0.25), 0 4px 16px rgba(0,0,0,0.1);
  border: 1px solid rgba(59,108,247,0.08);
}

.bmc-icon {
  font-size: 3rem;
  margin-bottom: 0.85rem;
  filter: drop-shadow(0 4px 12px rgba(240,192,64,0.4));
}

.bmc-title {
  font-size: 1.25rem;
  font-weight: 800;
  color: #1a2035;
  margin-bottom: 0.55rem;
  letter-spacing: -0.2px;
}

.bmc-text {
  font-size: 0.88rem;
  color: #64748b;
  margin-bottom: 1.75rem;
  line-height: 1.6;
}

.bmc-actions {
  display: flex;
  flex-direction: column;
  gap: 0.7rem;
}

.bmc-coffee-btn {
  display: block;
  padding: 0.72rem 1rem;
  background: linear-gradient(135deg, #f0c040 0%, #f5d565 100%);
  color: #1a1200;
  border-radius: 10px;
  font-size: 0.95rem;
  font-weight: 700;
  font-family: inherit;
  text-decoration: none;
  box-shadow: 0 4px 14px rgba(240,192,64,0.4);
  transition: transform 0.15s, box-shadow 0.15s;
}

.bmc-coffee-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 6px 20px rgba(240,192,64,0.5);
}

.bmc-skip-btn {
  padding: 0.62rem 1rem;
  background: none;
  border: 1.5px solid #e2e8f0;
  border-radius: 10px;
  font-size: 0.85rem;
  font-family: inherit;
  color: #64748b;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.15s;
}

.bmc-skip-btn:hover {
  background: #f7f9ff;
  border-color: #c7d7fc;
  color: #3b6cf7;
}
</style>
