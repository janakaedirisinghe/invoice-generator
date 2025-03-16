<template>
  <div class="max-w-4xl mx-auto p-5" :style="{ '--accent-color': selectedColor }">
    <div class="flex justify-between items-center mb-5 bg-white p-4 rounded-lg shadow-sm">
      <div class="flex items-center gap-3">
        <!-- Color Picker -->
        <span>Invoice Color:</span>
        <div class="flex gap-2">
          <button v-for="color in colorPalette" :key="color" :style="{ backgroundColor: color }"
            @click="selectedColor = color" :class="[
    'w-6 h-6 rounded-full transition-transform duration-200',
    { 'transform scale-110 border-2 border-gray-700': selectedColor === color }
  ]">
          </button>
        </div>
        <!-- Font Selector -->
        <div class="flex items-center gap-3">
          <span>Font:</span>
          <select v-model="selectedFont" class="border border-gray-300 rounded px-2 py-1">
            <option v-for="font in fonts" :key="font.value" :value="font.value">
              {{ font.label }}
            </option>
          </select>
        </div>
      </div>
      <button @click="downloadPDF"
        class="px-4 py-2 rounded-md font-medium transition-opacity duration-200 hover:opacity-90 text-white"
        :style="{ backgroundColor: selectedColor }">
        Download PDF
      </button>
    </div>

    <div :class="[selectedFont, 'bg-white p-10 rounded-lg shadow-lg w-full aspect-[1/1.414]']" ref="invoiceElement">
      <div class="flex justify-between mb-8 pb-5 border-b-2" :style="{ borderBottomColor: selectedColor }">
        <div class="company-details">
          <h1 contenteditable="true" @blur="updateData('companyName', $event)" class="text-2xl font-bold mb-3"
            :style="{ color: selectedColor }">
            {{ invoice.companyName }}
          </h1>
          <p contenteditable="true" @blur="updateData('companyAddress', $event)" class="text-gray-600">
            {{ invoice.companyAddress }}
          </p>
          <p contenteditable="true" @blur="updateData('companyContact', $event)" class="text-gray-600">
            {{ invoice.companyContact }}
          </p>
        </div>
        <div class="text-right">
          <h2 class="text-3xl font-bold mb-3" :style="{ color: selectedColor }">INVOICE</h2>
          <div class="flex flex-col gap-1">
            <div>
              <strong>Invoice #:</strong>
              <span contenteditable="true" @blur="updateData('invoiceNumber', $event)">
                {{ invoice.invoiceNumber }}
              </span>
            </div>
            <div>
              <strong>Date:</strong>
              <span contenteditable="true" @blur="updateData('invoiceDate', $event)">
                {{ invoice.invoiceDate }}
              </span>
            </div>
            <div>
              <strong>Due Date:</strong>
              <span contenteditable="true" @blur="updateData('dueDate', $event)">
                {{ invoice.dueDate }}
              </span>
            </div>
          </div>
        </div>
      </div>

      <div class="mb-8">
        <h3 class="text-lg font-semibold mb-2" :style="{ color: selectedColor }">Bill To:</h3>
        <p contenteditable="true" @blur="updateData('clientName', $event)" class="font-medium">
          {{ invoice.clientName }}
        </p>
        <p contenteditable="true" @blur="updateData('clientAddress', $event)" class="text-gray-600">
          {{ invoice.clientAddress }}
        </p>
        <p contenteditable="true" @blur="updateData('clientContact', $event)" class="text-gray-600">
          {{ invoice.clientContact }}
        </p>
      </div>

      <div class="mb-8">
        <table class="w-full">
          <thead>
            <tr :style="{ backgroundColor: selectedColor }" class="text-white">
              <th class="p-3 text-left">Item</th>
              <th class="p-3 text-left">Description</th>
              <th class="p-3 text-left">Quantity</th>
              <th class="p-3 text-left">Rate</th>
              <th class="p-3 text-left">Amount</th>
              <th class="p-3 w-10"></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in invoice.items" :key="index" class="border-b border-gray-100">
              <td contenteditable="true" @blur="updateItem(index, 'name', $event)" class="p-3">
                {{ item.name }}
              </td>
              <td contenteditable="true" @blur="updateItem(index, 'description', $event)" class="p-3">
                {{ item.description }}
              </td>
              <td contenteditable="true" @blur="updateItem(index, 'quantity', $event)" class="p-3 text-right">
                {{ item.quantity }}
              </td>
              <td contenteditable="true" @blur="updateItem(index, 'rate', $event)" class="p-3 text-right">
                {{ item.rate }}
              </td>
              <td class="p-3 text-right">{{ formatCurrency(item.quantity * item.rate) }}</td>
              <td class="p-3 text-center">
                <button @click="removeItem(index)"
                  class="w-6 h-6 rounded-full bg-red-500 text-white flex items-center justify-center">
                  ×
                </button>
              </td>
            </tr>
          </tbody>
          <tfoot>
            <tr>
              <td colspan="6" class="py-2">
                <button @click="addItem"
                  class="w-full py-2 px-3 bg-gray-100 border border-dashed border-gray-300 text-gray-500 rounded-md hover:bg-gray-200 transition-colors duration-200">
                  + Add Item
                </button>
              </td>
            </tr>
            <tr>
              <td colspan="3"></td>
              <td class="p-3 text-right"><strong>Subtotal:</strong></td>
              <td class="p-3 text-right">{{ formatCurrency(calculateSubtotal()) }}</td>
              <td></td>
            </tr>
            <tr>
              <td colspan="3"></td>
              <td class="p-3 text-right">
                <span contenteditable="true" @blur="updateData('taxLabel', $event)">{{ invoice.taxLabel
                  }}</span>:
              </td>
              <td class="p-3 text-right">
                <span contenteditable="true" @blur="updateTaxRate($event)">{{ invoice.taxRate
                  }}</span>%:
                {{ formatCurrency(calculateTax()) }}
              </td>
              <td></td>
            </tr>
            <tr>
              <td colspan="3"></td>
              <td class="p-3 text-right border-t-2" :style="{ borderTopColor: selectedColor }">
                <strong>Total:</strong>
              </td>
              <td class="p-3 text-right font-bold border-t-2" :style="{ borderTopColor: selectedColor }">
                {{ formatCurrency(calculateTotal()) }}
              </td>
              <td :style="{ borderTopColor: selectedColor }" class="border-t-2"></td>
            </tr>
          </tfoot>
        </table>
      </div>

      <div class="mb-8">
        <h3 class="text-lg font-semibold mb-2" :style="{ color: selectedColor }">Notes:</h3>
        <p contenteditable="true" @blur="updateData('notes', $event)" class="text-gray-600">
          {{ invoice.notes }}
        </p>
      </div>

      <div class="text-center text-gray-500 text-sm">
        <p contenteditable="true" @blur="updateData('paymentTerms', $event)">
          {{ invoice.paymentTerms }}
        </p>
        <p contenteditable="true" @blur="updateData('footer', $event)">
          {{ invoice.footer }}
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, reactive, onMounted } from 'vue'
import html2pdf from 'html2pdf.js'

export default {
  name: 'App',
  setup() {
    const invoiceElement = ref(null)
    const selectedColor = ref('#4f46e5') // Default color

    const selectedFont = ref('font-fira') // Default font
    const fonts = [
      { value: 'font-fira', label: 'Fira Code' },
      { value: 'font-inter', label: 'Inter' },
      { value: 'font-roboto', label: 'Roboto' },
      { value: 'font-poppins', label: 'Poppins' },
      { value: 'font-playfair', label: 'Playfair Display' }
    ]

    const colorPalette = [
      '#4f46e5', // Indigo
      '#0ea5e9', // Sky blue
      '#10b981', // Emerald
      '#f59e0b', // Amber
      '#ef4444', // Red
      '#8b5cf6', // Violet
      '#6366f1', // Blue
      '#000000'  // Black
    ]

    const invoice = reactive({
      companyName: 'Your Company Name',
      companyAddress: '123 Business Street, City, Country',
      companyContact: 'email@example.com | +1 (555) 123-4567',
      invoiceNumber: 'INV-001',
      invoiceDate: new Date().toLocaleDateString(),
      dueDate: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000).toLocaleDateString(),
      clientName: 'Client Name',
      clientAddress: 'Client Address, City, Country',
      clientContact: 'client@example.com | +1 (555) 987-6543',
      items: [
        {
          name: 'Service Item',
          description: 'Professional services rendered',
          quantity: 1,
          rate: 100
        }
      ],
      taxLabel: 'Tax',
      taxRate: 10,
      notes: 'Thank you for your business!',
      paymentTerms: 'Payment due within 30 days of receipt.',
      footer: '© 2025 Your Company Name. All rights reserved.'
    })

    const updateData = (field, event) => {
      invoice[field] = event.target.innerText
    }

    const updateItem = (index, field, event) => {
      let value = event.target.innerText

      // Convert to number for quantity and rate fields
      if (field === 'quantity' || field === 'rate') {
        value = parseFloat(value) || 0
      }

      invoice.items[index][field] = value
    }

    const updateTaxRate = (event) => {
      invoice.taxRate = parseFloat(event.target.innerText) || 0
    }

    const addItem = () => {
      invoice.items.push({
        name: 'New Item',
        description: 'Description',
        quantity: 1,
        rate: 0
      })
    }

    const removeItem = (index) => {
      invoice.items.splice(index, 1)
      if (invoice.items.length === 0) {
        addItem()
      }
    }

    const calculateSubtotal = () => {
      return invoice.items.reduce((total, item) => {
        return total + (item.quantity * item.rate)
      }, 0)
    }

    const calculateTax = () => {
      return calculateSubtotal() * (invoice.taxRate / 100)
    }

    const calculateTotal = () => {
      return calculateSubtotal() + calculateTax()
    }

    const formatCurrency = (amount) => {
      return new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency: 'USD'
      }).format(amount)
    }

    const downloadPDF = () => {
      const element = invoiceElement.value
      const opt = {
        margin: 10,
        filename: `Invoice-${invoice.invoiceNumber}.pdf`,
        image: { type: 'jpeg', quality: 0.98 },
        html2canvas: { scale: 2 },
        jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
      }

      // Create a clone of the invoice element
      const clone = element.cloneNode(true)

      // Apply the selected color directly to elements
      const colorElements = clone.querySelectorAll('[style*="--accent-color"]');
      colorElements.forEach(el => {
        const style = el.getAttribute('style');
        if (style.includes('--accent-color')) {
          el.setAttribute('style', style.replace('--accent-color', 'var(--accent-color)'));
        }
      });

      // Apply the selected color directly to specific elements
      const headings = clone.querySelectorAll('h1, h2, h3');
      headings.forEach(heading => {
        heading.style.color = selectedColor.value;
      });

      const tableHead = clone.querySelector('thead tr');
      if (tableHead) tableHead.style.backgroundColor = selectedColor.value;

      const borderElements = clone.querySelectorAll('.border-t-2, .border-b-2');
      borderElements.forEach(el => {
        el.style.borderColor = selectedColor.value;
      });

      // Make sure the font is preserved in the PDF
      clone.style.fontFamily = window.getComputedStyle(element).fontFamily;

      // Remove action buttons from the clone
      const actionButtons = clone.querySelectorAll('button, select')
      actionButtons.forEach(button => {
        button.style.display = 'none'
      })

      html2pdf().set(opt).from(clone).save()
    }

    return {
      selectedFont,
      fonts,
      invoiceElement,
      invoice,
      colorPalette,
      selectedColor,
      updateData,
      updateItem,
      updateTaxRate,
      addItem,
      removeItem,
      calculateSubtotal,
      calculateTax,
      calculateTotal,
      formatCurrency,
      downloadPDF
    }
  }
}
</script>

<style>
/* Some styles need to be kept for the dynamic color functionality */
:root {
  --accent-color: #4f46e5;
}

/* Any additional styles not covered by Tailwind */
[contenteditable="true"]:hover {
  outline: 1px dashed #ddd;
  padding: 2px;
}

[contenteditable="true"]:focus {
  outline: 2px solid var(--accent-color);
  padding: 2px;
}
</style>