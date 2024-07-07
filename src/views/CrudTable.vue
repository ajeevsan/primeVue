<template>
    <section>
        <div class="card">
            <Toolbar class="mb-4">
                <template #start>
                    <Button label="New" icon="pi pi-plus" class="p-button-success mr-2" @click="openNew" />
                    <Button label="Delete" icon="pi pi-trash" class="p-button-danger" @click="confirmDeleteSelected" :disabled="!selectedProducts || !selectedProducts.length" />
                </template>

                <template #end>
                    <FileUpload mode="basic" accept="image/*" :maxFileSize="1000000" label="Import" chooseLabel="Import" class="mr-2 inline-block" />
                    <Button label="Export" icon="pi pi-upload" class="p-button-help" @click="exportCSV($event)"  />
                </template>
            </Toolbar>

            <DataTable ref="dt" :value="products" :selection.sync="selectedProducts" dataKey="id"
                :paginator="true" :rows="10" :filters="filters"
                paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown" :rowsPerPageOptions="[5,10,25]"
                currentPageReportTemplate="Showing {first} to {last} of {totalRecords} products" responsiveLayout="scroll" showGridlines>
                <template #header>
                    <div class="table-header flex flex-column md:flex-row md:justify-content-between">
                        <h3 class="mb-2 md:m-0 md:align-self-center">Manage Project</h3>
                        <span class="p-input-icon-left">
                            <i class="pi pi-search" />
                            <InputText v-model="filters['global'].value" placeholder="Search..." />
                        </span>
                    </div>
                </template>

                <ColumnGroup type="header">
                    <Row>
                        <Column rowspan="2" selectionMode="multiple" :style="{width: '3rem'}" :exportable="false"></Column>
                        <Column rowspan="2" field="product" header="Product" :sortable="true" :style="{'min-width':'12rem'}"></Column>
                        <Column rowspan="2" field="oem" header="OEM" :sortable="true" :style="{'min-width':'16rem'}"></Column>
                        <Column rowspan="2" field="project" header="Project" :sortable="true" :style="{'min-width':'16rem'}"></Column>
                        <Column colspan="2" header="Primary Indicator" :style="{'min-width':'10rem'}"></Column>
                        <Column colspan="2" header="Secondary Indicator" :style="{'min-width':'10rem'}"></Column>
                        <Column rowspan="2" field="automation_validation" header="Automation Validation Issue Detection Portion" :sortable="true" :style="{'min-width':'10rem'}"></Column>
                        <Column rowspan="2" field="issue_trend" header="Issue Trend" :sortable="true" :style="{'min-width':'10rem'}"></Column>
                        <Column rowspan="2" header="Action" :exportable="false" :style="{'min-width':'8rem'}"></Column>
                    </Row>
                    <Row>
                        <Column field="primary_indicator_true_issue_rate" header="Automation Reliability True(not False) Issue Modified Reflection Rate" :sortable="true" :style="{'min-width':'12rem'}"></Column>
                        <Column field="primary_indicator_detection_rate" header="Automation Reliability True Issue Detection Rate" :sortable="true" :style="{'min-width':'12rem'}"></Column>
                        <Column field="secondary_indicator_true_issue_rate" header="Automation Issue Modified Reflection Rate" :sortable="true" :style="{'min-width':'12rem'}"></Column>
                        <Column field="secondary_indicator_detection_rate" header="Automation Issue Detection Rate" :sortable="true" :style="{'min-width':'12rem'}"></Column>
                    </Row>
                </ColumnGroup>

                <Column selectionMode="multiple" :style="{width: '3rem'}" :exportable="false"></Column>
                <Column field="product" header="Product" :sortable="true" :style="{'min-width':'12rem'}"></Column>
                <Column field="oem" header="OEM" :sortable="true" :style="{'min-width':'16rem'}"></Column>
                <Column field="project" header="Project" :sortable="true" :style="{'min-width':'16rem'}"></Column>
                <Column field="primary_indicator_true_issue_rate" header="Automation Reliability True(not False) Issue Modified Reflection Rate" :sortable="true" :style="{'min-width':'12rem'}"></Column>
                <Column field="primary_indicator_detection_rate" header="Automation Reliability True Issue Detection Rate" :sortable="true" :style="{'min-width':'12rem'}"></Column>
                <Column field="secondary_indicator_true_issue_rate" header="Automation Issue Modified Reflection Rate" :sortable="true" :style="{'min-width':'12rem'}"></Column>
                <Column field="secondary_indicator_detection_rate" header="Automation Issue Detection Rate" :sortable="true" :style="{'min-width':'12rem'}"></Column>
                <Column field="automation_validation" header="Automation Validation Issue Detection Portion" :sortable="true" :style="{'min-width':'10rem'}"></Column>
                <Column field="issue_trend" header="Issue Trend" :sortable="true" :style="{'min-width':'10rem'}"></Column>
                <Column header="Action" :exportable="false" :style="{'min-width':'8rem'}">
                    <template #body="slotProps">
                        <Button icon="pi pi-pencil" class="p-button-rounded p-button-success mr-2" @click="editProduct(slotProps.data)" />
                        <Button icon="pi pi-trash" class="p-button-rounded p-button-warning" @click="confirmDeleteProduct(slotProps.data)" />
                    </template>
                </Column>
            </DataTable>
        </div>

        <Dialog :visible.sync="productDialog" header="Product Details" :modal="true" class="p-fluid">
            <div class="field">
                <label for="product">Product</label>
                <InputText id="product" v-model="product.product" required="true" autofocus :class="{'p-invalid': submitted && !product.product}" />
                <small class="p-invalid" v-if="submitted && !product.product">Product is required.</small>
            </div>
            <div class="field">
                <label for="oem">OEM</label>
                <InputText id="oem" v-model="product.oem" required="true" :class="{'p-invalid': submitted && !product.oem}" />
                <small class="p-invalid" v-if="submitted && !product.oem">OEM is required.</small>
            </div>
            <div class="field">
                <label for="oem">Project</label>
                <InputText id="oem" v-model="product.project" required="true" :class="{'p-invalid': submitted && !product.oem}" />
                <small class="p-invalid" v-if="submitted && !product.oem">Project is required.</small>
            </div>
            <div class="field">
                <label for="primary_indicator_true_issue_rate">Primary Indicator True Issue Rate</label>
                <InputNumber id="primary_indicator_true_issue_rate" v-model="product.primary_indicator_true_issue_rate" />
            </div>
            <div class="field">
                <label for="primary_indicator_detection_rate">Primary Indicator Detection Rate</label>
                <InputNumber id="primary_indicator_detection_rate" v-model="product.primary_indicator_detection_rate" />
            </div>
            <div class="field">
                <label for="secondary_indicator_true_issue_rate">Secondary Indicator True Issue Rate</label>
                <InputNumber id="secondary_indicator_true_issue_rate" v-model="product.secondary_indicator_true_issue_rate" />
            </div>
            <div class="field">
                <label for="secondary_indicator_detection_rate">Secondary Indicator Detection Rate</label>
                <InputNumber id="secondary_indicator_detection_rate" v-model="product.secondary_indicator_detection_rate" />
            </div>
            <div class="field">
                <label for="automation_validation">Automation Validation Issue Detection Portion</label>
                <InputText id="automation_validation" v-model="product.automation_validation" />
            </div>
            <div class="field">
                <label for="issue_trend">Issue Trend</label>
                <InputText id="issue_trend" v-model="product.issue_trend" />
            </div>
            <template #footer>
                <Button label="Cancel" icon="pi pi-times" class="p-button-text" @click="hideDialog" />
                <Button label="Save" icon="pi pi-check" class="p-button-text" @click="saveProduct" />
            </template>
        </Dialog>


        <Dialog :visible.sync="deleteProductDialog" :style="{width: '450px'}" header="Confirm" :modal="true">
            <div class="confirmation-content">
                <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                <span v-if="product">Are you sure you want to delete <b>{{product.name}}</b>?</span>
            </div>
            <template #footer>
                <Button label="No" icon="pi pi-times" class="p-button-text" @click="deleteProductDialog = false"/>
                <Button label="Yes" icon="pi pi-check" class="p-button-text" @click="deleteProduct" />
            </template>
        </Dialog>

        <Dialog :visible.sync="deleteProductsDialog" :style="{width: '450px'}" header="Confirm" :modal="true">
            <div class="confirmation-content">
                <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                <span v-if="product">Are you sure you want to delete the selected products?</span>
            </div>
            <template #footer>
                <Button label="No" icon="pi pi-times" class="p-button-text" @click="deleteProductsDialog = false"/>
                <Button label="Yes" icon="pi pi-check" class="p-button-text" @click="deleteSelectedProducts" />
            </template>
        </Dialog>
    </section>
</template>

<script>
import { FilterMatchMode } from 'primevue/api';
import DataTable from 'primevue/datatable';
import Column from 'primevue/column';
import Button from 'primevue/button';
import Dialog from 'primevue/dialog';
import InputText from 'primevue/inputtext';
import InputNumber from 'primevue/inputnumber';
import Toolbar from 'primevue/toolbar';
import FileUpload from 'primevue/fileupload';
import ColumnGroup from 'primevue/columngroup';
import Row from 'primevue/row';

export default {
    name: 'CrudTable',
    components: {
        DataTable,
        Column,
        Button,
        Dialog,
        InputText,
        InputNumber,
        Toolbar,
        FileUpload,
        ColumnGroup,
        Row
    },
    data() {
        return {
            products: [],
            productDialog: false,
            deleteProductDialog: false,
            deleteProductsDialog: false,
            product: {},
            selectedProducts: null,
            filters: {},
            submitted: false,
            statuses: [
                { label: 'INSTOCK', value: 'instock' },
                { label: 'LOWSTOCK', value: 'lowstock' },
                { label: 'OUTOFSTOCK', value: 'outofstock' }
            ]
        }
    },
    created() {
        this.initFilters();
    },
    mounted() {
        // Dummy data
        this.products = [
                {
                    id: 1,
                    product: 'Product A',
                    oem: 'OEM A',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 95,
                    primary_indicator_detection_rate: 90,
                    secondary_indicator_true_issue_rate: 85,
                    secondary_indicator_detection_rate: 80,
                    automation_validation: '80%',
                    issue_trend: 'Stable'
                },
                {
                    id: 2,
                    product: 'Product B',
                    oem: 'OEM B',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 93,
                    primary_indicator_detection_rate: 88,
                    secondary_indicator_true_issue_rate: 83,
                    secondary_indicator_detection_rate: 78,
                    automation_validation: '85%',
                    issue_trend: 'Increasing'
                },
                {
                    id: 3,
                    product: 'Product C',
                    oem: 'OEM C',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 97,
                    primary_indicator_detection_rate: 92,
                    secondary_indicator_true_issue_rate: 87,
                    secondary_indicator_detection_rate: 82,
                    automation_validation: '75%',
                    issue_trend: 'Decreasing'
                },
                {
                    id: 4,
                    product: 'Product D',
                    oem: 'OEM D',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 94,
                    primary_indicator_detection_rate: 89,
                    secondary_indicator_true_issue_rate: 84,
                    secondary_indicator_detection_rate: 79,
                    automation_validation: '70%',
                    issue_trend: 'Stable'
                },
                {
                    id: 5,
                    product: 'Product E',
                    oem: 'OEM E',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 96,
                    primary_indicator_detection_rate: 91,
                    secondary_indicator_true_issue_rate: 86,
                    secondary_indicator_detection_rate: 81,
                    automation_validation: '90%',
                    issue_trend: 'Increasing'
                },
                {
                    id: 1,
                    product: 'Product A',
                    oem: 'OEM A',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 95,
                    primary_indicator_detection_rate: 90,
                    secondary_indicator_true_issue_rate: 85,
                    secondary_indicator_detection_rate: 80,
                    automation_validation: '80%',
                    issue_trend: 'Stable'
                },
                {
                    id: 2,
                    product: 'Product B',
                    oem: 'OEM B',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 93,
                    primary_indicator_detection_rate: 88,
                    secondary_indicator_true_issue_rate: 83,
                    secondary_indicator_detection_rate: 78,
                    automation_validation: '85%',
                    issue_trend: 'Increasing'
                },
                {
                    id: 3,
                    product: 'Product C',
                    oem: 'OEM C',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 97,
                    primary_indicator_detection_rate: 92,
                    secondary_indicator_true_issue_rate: 87,
                    secondary_indicator_detection_rate: 82,
                    automation_validation: '75%',
                    issue_trend: 'Decreasing'
                },
                {
                    id: 4,
                    product: 'Product D',
                    oem: 'OEM D',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 94,
                    primary_indicator_detection_rate: 89,
                    secondary_indicator_true_issue_rate: 84,
                    secondary_indicator_detection_rate: 79,
                    automation_validation: '70%',
                    issue_trend: 'Stable'
                },
                {
                    id: 5,
                    product: 'Product E',
                    oem: 'OEM E',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 96,
                    primary_indicator_detection_rate: 91,
                    secondary_indicator_true_issue_rate: 86,
                    secondary_indicator_detection_rate: 81,
                    automation_validation: '90%',
                    issue_trend: 'Increasing'
                },
                {
                    id: 1,
                    product: 'Product A',
                    oem: 'OEM A',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 95,
                    primary_indicator_detection_rate: 90,
                    secondary_indicator_true_issue_rate: 85,
                    secondary_indicator_detection_rate: 80,
                    automation_validation: '80%',
                    issue_trend: 'Stable'
                },
                {
                    id: 2,
                    product: 'Product B',
                    oem: 'OEM B',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 93,
                    primary_indicator_detection_rate: 88,
                    secondary_indicator_true_issue_rate: 83,
                    secondary_indicator_detection_rate: 78,
                    automation_validation: '85%',
                    issue_trend: 'Increasing'
                },
                {
                    id: 3,
                    product: 'Product C',
                    oem: 'OEM C',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 97,
                    primary_indicator_detection_rate: 92,
                    secondary_indicator_true_issue_rate: 87,
                    secondary_indicator_detection_rate: 82,
                    automation_validation: '75%',
                    issue_trend: 'Decreasing'
                },
                {
                    id: 4,
                    product: 'Product D',
                    oem: 'OEM D',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 94,
                    primary_indicator_detection_rate: 89,
                    secondary_indicator_true_issue_rate: 84,
                    secondary_indicator_detection_rate: 79,
                    automation_validation: '70%',
                    issue_trend: 'Stable'
                },
                {
                    id: 5,
                    product: 'Product E',
                    oem: 'OEM E',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 96,
                    primary_indicator_detection_rate: 91,
                    secondary_indicator_true_issue_rate: 86,
                    secondary_indicator_detection_rate: 81,
                    automation_validation: '90%',
                    issue_trend: 'Increasing'
                },
                {
                    id: 1,
                    product: 'Product A',
                    oem: 'OEM A',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 95,
                    primary_indicator_detection_rate: 90,
                    secondary_indicator_true_issue_rate: 85,
                    secondary_indicator_detection_rate: 80,
                    automation_validation: '80%',
                    issue_trend: 'Stable'
                },
                {
                    id: 2,
                    product: 'Product B',
                    oem: 'OEM B',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 93,
                    primary_indicator_detection_rate: 88,
                    secondary_indicator_true_issue_rate: 83,
                    secondary_indicator_detection_rate: 78,
                    automation_validation: '85%',
                    issue_trend: 'Increasing'
                },
                {
                    id: 3,
                    product: 'Product C',
                    oem: 'OEM C',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 97,
                    primary_indicator_detection_rate: 92,
                    secondary_indicator_true_issue_rate: 87,
                    secondary_indicator_detection_rate: 82,
                    automation_validation: '75%',
                    issue_trend: 'Decreasing'
                },
                {
                    id: 4,
                    product: 'Product D',
                    oem: 'OEM D',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 94,
                    primary_indicator_detection_rate: 89,
                    secondary_indicator_true_issue_rate: 84,
                    secondary_indicator_detection_rate: 79,
                    automation_validation: '70%',
                    issue_trend: 'Stable'
                },
                {
                    id: 5,
                    product: 'Product E',
                    oem: 'OEM E',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 96,
                    primary_indicator_detection_rate: 91,
                    secondary_indicator_true_issue_rate: 86,
                    secondary_indicator_detection_rate: 81,
                    automation_validation: '90%',
                    issue_trend: 'Increasing'
                },
                {
                    id: 1,
                    product: 'Product A',
                    oem: 'OEM A',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 95,
                    primary_indicator_detection_rate: 90,
                    secondary_indicator_true_issue_rate: 85,
                    secondary_indicator_detection_rate: 80,
                    automation_validation: '80%',
                    issue_trend: 'Stable'
                },
                {
                    id: 2,
                    product: 'Product B',
                    oem: 'OEM B',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 93,
                    primary_indicator_detection_rate: 88,
                    secondary_indicator_true_issue_rate: 83,
                    secondary_indicator_detection_rate: 78,
                    automation_validation: '85%',
                    issue_trend: 'Increasing'
                },
                {
                    id: 3,
                    product: 'Product C',
                    oem: 'OEM C',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 97,
                    primary_indicator_detection_rate: 92,
                    secondary_indicator_true_issue_rate: 87,
                    secondary_indicator_detection_rate: 82,
                    automation_validation: '75%',
                    issue_trend: 'Decreasing'
                },
                {
                    id: 4,
                    product: 'Product D',
                    oem: 'OEM D',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 94,
                    primary_indicator_detection_rate: 89,
                    secondary_indicator_true_issue_rate: 84,
                    secondary_indicator_detection_rate: 79,
                    automation_validation: '70%',
                    issue_trend: 'Stable'
                },
                {
                    id: 5,
                    product: 'Product E',
                    oem: 'OEM E',
                    project: 'project 1',
                    primary_indicator_true_issue_rate: 96,
                    primary_indicator_detection_rate: 91,
                    secondary_indicator_true_issue_rate: 86,
                    secondary_indicator_detection_rate: 81,
                    automation_validation: '90%',
                    issue_trend: 'Increasing'
                },
            ]
    },
    methods: {
        formatCurrency(value) {
            if(value)
                return value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
            return;
        },
        openNew() {
            this.product = {};
            this.submitted = false;
            this.productDialog = true;
        },
        hideDialog() {
            this.productDialog = false;
            this.submitted = false;
        },
        saveProduct() {
            this.submitted = true;

            if (this.product.name.trim()) {
                if (this.product.id) {
                    this.product.inventoryStatus = this.product.inventoryStatus.value ? this.product.inventoryStatus.value: this.product.inventoryStatus;
                    this.products[this.findIndexById(this.product.id)] = this.product;
                    this.$toast.add({ severity: 'success', summary: 'Successful', detail: 'Product Updated', life: 3000 });
                }
                else {
                    this.product.id = this.createId();
                    this.product.code = this.createId();
                    this.product.image = 'product-placeholder.svg';
                    this.product.inventoryStatus = this.product.inventoryStatus ? this.product.inventoryStatus.value : 'INSTOCK';
                    this.products.push(this.product);
                    this.$toast.add({ severity: 'success', summary: 'Successful', detail: 'Product Created', life: 3000 });
                }

                this.productDialog = false;
                this.product = {};
            }
        },
        editProduct(product) {
            this.product = { ...product };
            this.productDialog = true;
        },
        confirmDeleteProduct(product) {
            this.product = product;
            this.deleteProductDialog = true;
        },
        deleteProduct() {
            this.products = this.products.filter(val => val.id !== this.product.id);
            this.deleteProductDialog = false;
            this.product = {};
            this.$toast.add({ severity: 'success', summary: 'Successful', detail: 'Product Deleted', life: 3000 });
        },
        findIndexById(id) {
            let index = -1;
            for (let i = 0; i < this.products.length; i++) {
                if (this.products[i].id === id) {
                    index = i;
                    break;
                }
            }

            return index;
        },
        createId() {
            let id = '';
            var chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
            for (var i = 0; i < 5; i++) {
                id += chars.charAt(Math.floor(Math.random() * chars.length));
            }
            return id;
        },
        exportCSV() {
            this.$refs.dt.exportCSV();
        },
        confirmDeleteSelected() {
            this.deleteProductsDialog = true;
        },
        deleteSelectedProducts() {
            this.products = this.products.filter(val => !this.selectedProducts.includes(val));
            this.deleteProductsDialog = false;
            this.selectedProducts = null;
            this.$toast.add({ severity: 'success', summary: 'Successful', detail: 'Products Deleted', life: 3000 });
        },
        initFilters() {
            this.filters = {
                'global': { value: null, matchMode: FilterMatchMode.CONTAINS },
            }
        }
    }
}
</script>

<style>
.table-header {
    display: flex;
    align-items: center;
    justify-content: space-between;

    @media screen and (max-width: 960px) {
        align-items: start;
    }
}

.product-image {
    width: 100px;
    box-shadow: 0 3px 6px rgba(0, 0, 0, 0.16), 0 3px 6px rgba(0, 0, 0, 0.23);
}

.p-dialog .product-image {
    width: 150px;
    margin: 0 auto 2rem auto;
    display: block;
}

.confirmation-content {
    display: flex;
    align-items: center;
    justify-content: center;
}

@media screen and (max-width: 960px) {
    :deep(.p-toolbar) {
        flex-wrap: wrap;

        .p-button {
            margin-bottom: 0.25rem;
        }
    }
}
</style>
