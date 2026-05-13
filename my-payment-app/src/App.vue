<template>
  <div class="fa-dashboard">
    <!-- Top Header -->
    <div class="fa-header-area">
      <h2 class="fa-page-title">Payment Method Definition</h2>
      <button class="fa-btn-save" @click="handleSave" :disabled="isLoading">
        {{ isLoading ? 'SAVING...' : 'SAVE CHANGES' }}
      </button>
    </div>

    <!-- Master Table -->
    <div class="fa-table-card">
      <table class="fa-data-table">
        <thead>
          <tr>
            <th style="width: 80px; text-align: center;">Select</th>
            <th style="width: 250px;">Method</th>
            <th>Display Name</th>
            <th style="width: 150px; text-align: center;">Reversible</th>
            <th style="width: 100px; text-align: center;">Action</th>
          </tr>
          <tr class="fa-filter-row">
            <td></td>
            <td>
              <input type="text" v-model="searchQuery" placeholder="Filter Method..." class="fa-inline-filter" />
            </td>
            <td></td>
            <td></td>
            <td></td>
          </tr>
        </thead>
        <tbody>
          <tr v-for="method in filteredMethods" :key="method.id" :class="{ 'row-disabled': !method.subscribed }">
            <td class="text-center">
              <input type="checkbox" v-model="method.subscribed" @change="onToggleMethod(method)" class="fa-checkbox-custom" />
            </td>
            <td class="method-name-cell">{{ method.name }}</td>
            <td>
              <input 
                type="text" 
                v-model="method.displayName" 
                :disabled="!method.subscribed" 
                class="fa-table-input"
              />
            </td>
            <td class="text-center">
              <input type="checkbox" v-model="method.isReversible" :disabled="!method.subscribed" class="fa-checkbox-custom" />
            </td>
            <td class="text-center">
              <button class="fa-btn-gear" @click="openDrawer(method)" :disabled="!method.subscribed">
                <svg viewBox="0 0 24 24" width="18" height="18"><path fill="currentColor" d="M12,15.5A3.5,3.5 0 0,1 8.5,12A3.5,3.5 0 0,1 12,8.5A3.5,3.5 0 0,1 15.5,12A3.5,3.5 0 0,1 12,15.5M19.43,12.97C19.47,12.65 19.5,12.33 19.5,12C19.5,11.67 19.47,11.35 19.43,11.03L21.54,9.37C21.73,9.22 21.78,8.95 21.66,8.73L19.66,5.27C19.54,5.05 19.27,4.96 19.05,5.05L16.56,6.05C16.04,5.66 15.47,5.32 14.87,5.07L14.5,2.42C14.46,2.18 14.25,2 14,2H10C9.75,2 9.54,2.18 9.5,2.42L9.13,5.07C8.53,5.32 7.96,5.66 7.44,6.05L4.95,5.05C4.73,4.96 4.46,5.05 4.34,5.27L2.34,8.73C2.22,8.95 2.27,9.22 2.46,9.37L4.57,11.03C4.53,11.35 4.5,11.67 4.5,12C4.5,12.33 4.53,12.65 4.57,12.97L2.46,14.63C2.27,14.78 2.22,15.05 2.34,15.27L4.34,18.73C4.46,18.95 4.73,19.04 4.95,18.95L7.44,17.95C7.96,18.34 8.53,18.68 9.13,18.93L9.5,21.58C9.54,21.82 9.75,22 10,22H14C14.25,22 14.46,21.82 14.5,21.58L14.87,18.93C15.47,18.68 16.04,18.34 16.56,17.95L19.05,18.95C19.27,19.04 19.54,18.95 19.66,18.73L21.66,15.27C21.78,15.05 21.73,14.78 21.54,14.63L19.43,12.97Z" /></svg>
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Side Configuration Drawer -->
    <div class="fa-drawer-mask" v-if="isDrawerOpen" @click.self="closeDrawer">
      <div class="fa-side-drawer-container">
        <div class="drawer-header">
          <h3 class="drawer-header-text">Configure: {{ activeMethod?.name }}</h3>
        </div>
        
        <div class="drawer-body-scrollable">
          <div v-for="field in activeMethod?.fields" :key="field.fieldType" class="drawer-form-item">
            <label class="drawer-field-label">{{ getFieldTypeName(field.fieldType) }}</label>
            <input type="text" v-model="field.displayName" class="drawer-input-underline" />
            
            <div class="drawer-radio-group">
              <label class="radio-item"><input type="radio" :value="0" v-model="field.state"> Hidden</label>
              <label class="radio-item"><input type="radio" :value="1" v-model="field.state"> Optional</label>
              <label class="radio-item"><input type="radio" :value="2" v-model="field.state"> Mandatory</label>
            </div>
          </div>
        </div>
        
        <div class="drawer-footer-action">
          <button class="fa-btn-submit" @click="closeDrawer">DONE</button>
        </div>
      </div>
    </div>

    <!-- Success Toast -->
    <div v-if="toast.show" class="v-toast v-toast--bottom" style="z-index: 999999;">
      <div class="fa-toast-success-box">
        <svg viewBox="0 0 24 24" width="20" height="20"><path fill="currentColor" d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z"/></svg>
        <span>{{ toast.message }}</span>
      </div>
    </div>

    <!-- Confirmation Modal -->
    <div class="fa-modal-overlay" v-if="showModal">
      <div class="fa-modal-box">
        <div class="modal-header-section"><h2>Deactivate Method</h2></div>
        <div class="modal-body-section">
          <p>Are you sure you want to deactivate <strong>{{ methodToToggle?.name }}</strong>? Field users will no longer see this as a payment option.</p>
        </div>
        <div class="modal-footer-section">
          <button class="btn-cancel" @click="cancelToggle">Cancel</button>
          <button class="btn-cyan-confirm" @click="confirmToggle">Confirm</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import axios from 'axios'; // You may need to run `npm install axios` in your terminal

const searchQuery = ref('');
const isDrawerOpen = ref(false);
const showModal = ref(false);
const isLoading = ref(false);
const activeMethod = ref(null);
const methodToToggle = ref(null);
const toast = ref({ show: false, message: '' });

// Current Context (Usually pulled from Vuex/Pinia store in production)
const COMPANY_ID = 1; 

const getFieldTypeName = (type) => ["Remarks", "Payment Proof", "Bank Selection", "Transaction Date"][type];

// Base Definition of all possible methods. We will overwrite these with DB data on load.
const methods = ref([
  { id: 0, name: 'Cash', subscribed: false, displayName: 'Cash', isReversible: false, fields: [{ fieldType: 0, displayName: 'Payment Details', state: 1 }, { fieldType: 1, displayName: 'Payment Proof', state: 1 }] },
  { id: 1, name: 'Cheque', subscribed: false, displayName: 'Cheque', isReversible: true, fields: [{ fieldType: 0, displayName: 'Cheque Number', state: 2 }, { fieldType: 2, displayName: 'Bank Name', state: 2 }, { fieldType: 3, displayName: 'Cheque Date', state: 2 }] },
  { id: 2, name: 'UPI Payment', subscribed: false, displayName: 'UPI Payment', isReversible: false, fields: [{ fieldType: 0, displayName: 'Transaction ID', state: 2 }] },
  { id: 24, name: 'MTN', subscribed: false, displayName: 'MTN Money', isReversible: false, fields: [] }
]);

const filteredMethods = computed(() => methods.value.filter(m => m.name.toLowerCase().includes(searchQuery.value.toLowerCase())));

const showSuccess = (msg) => {
  toast.value = { show: true, message: msg };
  setTimeout(() => { toast.value.show = false; }, 3000);
};

// ==========================================
// BACKEND LOGIC: FETCHING DATA ON PAGE LOAD
// ==========================================
onMounted(async () => {
  try {
    // API Call to get existing subscriptions for this company
    // Replace with your actual FieldAssist endpoint: e.g., `/api/CompanyWisePaymentMethodSubscription/GetAll?companyId=${COMPANY_ID}`
    const response = await axios.get(`YOUR_GET_API_ENDPOINT_HERE`); 
    
    const savedData = response.data; // Array of saved payment methods
    
    // Map database data back to UI
    savedData.forEach(savedItem => {
      const match = methods.value.find(m => m.id === savedItem.PaymentMethod);
      if (match) {
        match.subscribed = true;
        match.displayName = savedItem.DisplayName;
        match.isReversible = savedItem.IsReversible;
        
        // Parse the PropertyJson back into UI "state" (0, 1, or 2)
        if (savedItem.PropertyJson) {
          const properties = JSON.parse(savedItem.PropertyJson);
          properties.forEach(prop => {
            const fieldMatch = match.fields.find(f => f.fieldType === prop.FieldType);
            if (fieldMatch) {
              fieldMatch.displayName = prop.DisplayName;
              // Translate DB Boolean to UI State
              if (!prop.IsVisible) fieldMatch.state = 0; // Hidden
              else if (prop.IsVisible && !prop.IsMandatory) fieldMatch.state = 1; // Optional
              else if (prop.IsVisible && prop.IsMandatory) fieldMatch.state = 2; // Mandatory
            }
          });
        }
      }
    });
  } catch (error) {
    console.warn("Using default layout. Backend GET endpoint not yet connected.");
  }
});

// ==========================================
// BACKEND LOGIC: FORMATTING & SAVING DATA
// ==========================================
const handleSave = async () => {
  isLoading.value = true;
  
  // 1. Create the payload array matching your exact SQL Schema
  const payload = methods.value.filter(m => m.subscribed).map(m => {
    
    // 2. Format PropertyJson
    let propertyJsonString = null;
    if (m.fields && m.fields.length > 0) {
      const propertiesArray = m.fields.map(f => {
        return {
          FieldType: f.fieldType,
          DisplayName: f.displayName,
          IsVisible: f.state > 0, // True if state is 1 or 2
          IsMandatory: f.state === 2 // True only if state is 2
        };
      });
      propertyJsonString = JSON.stringify(propertiesArray);
    }

    // 3. Construct the row object
    return {
      CompanyId: COMPANY_ID,
      PaymentMethod: m.id,
      DisplayName: m.displayName,
      IsReversible: m.isReversible,
      PropertyJson: propertyJsonString,
      UpdationContext: "WebAdmin_PaymentDef", // Identifies where the change came from
      UpdatedAt: new Date().toISOString()
    };
  });

  try {
    // 4. Send to Backend
    // Replace with your actual FieldAssist POST endpoint
    console.log("PAYLOAD READY FOR BACKEND:", payload); // Check your browser console to see the perfectly formatted data!
    await axios.post('YOUR_POST_API_ENDPOINT_HERE', payload);
    
    showSuccess("Configurations saved successfully!");
  } catch (error) {
    console.error("Error saving data:", error);
    // Even if API fails locally, we show success for the prototype demo
    showSuccess("Configurations mocked as saved! (Check Console)"); 
  } finally {
    isLoading.value = false;
  }
};

// --- UI Interaction Logic ---
const onToggleMethod = (method) => {
  if (!method.subscribed) {
    methodToToggle.value = method;
    showModal.value = true;
    method.subscribed = true;
  }
};
const confirmToggle = () => { methodToToggle.value.subscribed = false; showModal.value = false; showSuccess("Method deactivated successfully"); };
const cancelToggle = () => { showModal.value = false; };
const openDrawer = (method) => { activeMethod.value = method; isDrawerOpen.value = true; };
const closeDrawer = () => isDrawerOpen.value = false;
</script>

<style scoped>
/* Main Page Styles */
.fa-dashboard { background: #f4f7f9; padding: 25px; font-family: 'Segoe UI', sans-serif; min-height: 100vh; }
.fa-header-area { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.fa-page-title { color: #444; font-weight: 600; font-size: 1.4rem; }
.fa-btn-save { background: #1a9245; color: white; border: none; padding: 10px 20px; border-radius: 4px; font-weight: bold; cursor: pointer; transition: 0.3s;}
.fa-btn-save:disabled { opacity: 0.7; cursor: not-allowed; }

/* Table Styling */
.fa-table-card { background: white; border-radius: 4px; border: 1px solid #e0e6ed; overflow: hidden; }
.fa-data-table { width: 100%; border-collapse: collapse; table-layout: fixed; }
.fa-data-table th { background: #f8fafd; color: #337ab7; text-align: left; padding: 14px 20px; border-bottom: 1px solid #e0e6ed; font-size: 0.95rem; }
.fa-data-table td { padding: 12px 20px; border-bottom: 1px solid #f0f3f7; vertical-align: middle; text-align: left; }
.method-name-cell { color: #555; font-weight: 500; }
.row-disabled { opacity: 0.55; background: #fafafa; }
.fa-inline-filter, .fa-table-input { width: 100%; padding: 8px 12px; border: 1px solid #dcdfe6; border-radius: 4px; box-sizing: border-box; }
.fa-btn-gear { background: none; border: none; color: #337ab7; cursor: pointer; }

/* Side Drawer Styles */
.fa-drawer-mask { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.35); z-index: 1000; }
.fa-side-drawer-container { position: absolute; right: 0; top: 0; width: 450px; height: 100%; background: white; box-shadow: -5px 0 15px rgba(0,0,0,0.1); display: flex; flex-direction: column; text-align: left; }
.drawer-header { padding: 25px; border-bottom: 1px solid #f0f0f0; }
.drawer-header-text { margin: 0; color: #22a1d2; font-size: 1.2rem; }
.drawer-body-scrollable { flex: 1; padding: 30px; overflow-y: auto; overflow-x: hidden; }
.drawer-form-item { margin-bottom: 35px; }
.drawer-field-label { color: #999; font-size: 11px; text-transform: uppercase; font-weight: 700; display: block; margin-bottom: 8px; letter-spacing: 0.5px; }
.drawer-input-underline { width: 100%; background: transparent; border: none; border-bottom: 2px solid #eee; padding: 8px 4px; outline: none; transition: 0.3s; }
.drawer-input-underline:focus { border-bottom-color: #22a1d2; }
.drawer-radio-group { display: flex; gap: 20px; margin-top: 12px; }
.radio-item { display: flex; align-items: center; gap: 6px; cursor: pointer; color: #666; }
.drawer-footer-action { padding: 20px; border-top: 1px solid #f0f0f0; }
.fa-btn-submit { width: 100%; background: #16a34a; color: white; border: none; padding: 14px; border-radius: 4px; font-weight: bold; cursor: pointer; }

/* Toast & Modal */
.v-toast--bottom { position: fixed; bottom: 40px; left: 50%; transform: translateX(-50%); z-index: 2000; }
.fa-toast-success-box { background: #4caf50; color: white; padding: 12px 24px; border-radius: 4px; display: flex; align-items: center; gap: 10px; box-shadow: 0 4px 12px rgba(0,0,0,0.1); }
.fa-modal-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); display: flex; justify-content: center; align-items: center; z-index: 2000; }
.fa-modal-box { background: white; width: 500px; border-radius: 4px; overflow: hidden; }
.modal-header-section { padding: 20px; border-bottom: 1px solid #eee; }
.modal-body-section { padding: 25px; color: #666; }
.modal-footer-section { padding: 15px 20px; border-top: 1px solid #eee; display: flex; justify-content: flex-end; gap: 10px; }
.btn-cancel { background: white; border: 1px solid #ccc; padding: 8px 20px; border-radius: 4px; cursor: pointer; }
.btn-cyan-confirm { background: #00add8; color: white; border: none; padding: 8px 20px; border-radius: 4px; font-weight: bold; cursor: pointer; }
.text-center { text-align: center !important; }
</style>