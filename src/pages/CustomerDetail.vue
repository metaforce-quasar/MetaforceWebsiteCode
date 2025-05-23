<template>
    <div class="row q-mt-lg">
        <div class="col-1"></div>
        <div class="col-10 row">
            <div class="col-6 q-pa-sm">
                <q-card>
                    <q-toolbar class="bg-primary text-white">
                        <q-icon name="account_circle" color="white" size="md" />
                        <q-toolbar-title>Account Information</q-toolbar-title>
                        <q-btn @click="updateAccount" :loading="isAccountUpdating" icon="cloud_upload" class="bg-white text-primary" dense no-caps></q-btn>
                    </q-toolbar>
                    <q-card-section>
                        <q-form ref="formCmp">
                            <q-input v-model="customer.easymeta__Email__c" label="Email address" type="email" class="q-mb-md" disable outlined hide-bottom-space>
                                <template v-slot:prepend><q-icon name="email" /></template>
                            </q-input>
                            <q-input v-model="customer.easymeta__FirstName__c" label="First Name" class="q-mb-md" :rules="nameRules" lazy-rules="ondemand" outlined hide-bottom-space>
                                <template v-slot:prepend><q-icon name="person" /></template>
                            </q-input>
                            <q-input v-model="customer.easymeta__LastName__c" label="Last Name" class="q-mb-md" :rules="nameRules" lazy-rules="ondemand" outlined hide-bottom-space>
                                <template v-slot:prepend><q-icon name="person" /></template>
                            </q-input>
                        </q-form>
                    </q-card-section>
                </q-card>
            </div>

            <div class="col-6  q-pa-sm">
                <q-card>
                    <q-toolbar class="bg-primary text-white">
                        <q-icon name="contact_support" color="white" size="md" />
                        <q-toolbar-title>My Cases</q-toolbar-title>
                        <q-btn @click="$refs.newCasePopupCmp.show()" icon="add_circle_outline" class="bg-white text-primary" dense no-caps></q-btn>
                    </q-toolbar>
                    <q-card-section v-if="isCasesLoading" class="text-center">
                        <q-spinner-bars size="sm" color="primary"></q-spinner-bars>
                    </q-card-section>
                    <template v-else>
                        <q-markup-table v-if="myCases.length">
                            <thead>
                                <tr>
                                    <th class="text-left">Case #</th>
                                    <th class="text-left">Subject</th>
                                    <th class="text-left">Status</th>
                                    <th class="text-left">CreatedDate</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="(rec, index) in myCases" :key="index">
                                    <td class="text-left">
                                        <a @click="viewCaseDetail(rec)" class="text-underline">{{rec.CaseNumber}}</a>
                                    </td>
                                    <td class="text-left">{{rec.Subject}}</td>
                                    <td class="text-left">{{rec.Status}}</td>
                                    <td class="text-left">{{formatDatetime(rec.CreatedDate)}}</td>
                                </tr>
                            </tbody>
                        </q-markup-table>
                        <q-card-section v-else class="text-center q-py-xl">
                            You haven't raised any cases yet!
                        </q-card-section>
                    </template>

                </q-card>
            </div>

            <div class="col-12 q-pt-md">
                <q-card>
                    <q-toolbar class="bg-primary text-white">
                        <q-icon name="redeem" color="white" size="md" />
                        <q-toolbar-title>My Subscriptions</q-toolbar-title>
                        <q-btn v-if="hasActiveSubscription" icon="shopping_cart" href="https://customer-portal.paddle.com/cpl_01jtyvfqptck70qbppg3gmhsjw" target="_blank" label="Manage Subscription" class="bg-white text-primary" dense no-caps></q-btn>
                    </q-toolbar>

                    <q-markup-table v-if="subscriptions.length">
                        <thead>
                            <tr>
                                <th class="text-left">Name</th>
                                <th class="text-left">Status</th>
                                <th class="text-left">Started At</th>
                                <th class="text-left">Current Billing Starts At</th>
                                <th class="text-left">Current Billing Ends At</th>
                                <th class="text-left">Next Billed At</th>
                                <th class="text-left">Billing Cycle</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="(rec, index) in subscriptions" :key="index">
                                <td class="text-left">{{rec.Name}}</td>
                                <td class="text-left">{{rec.easymeta__Status__c}}</td>
                                <td class="text-left">{{formatDatetime(rec.easymeta__Started_At__c)}}</td>
                                <td class="text-left">{{formatDatetime(rec.easymeta__Current_Billing_Starts_At__c)}}</td>
                                <td class="text-left">{{formatDatetime(rec.easymeta__Current_Billing_Ends_At__c)}}</td>
                                <td class="text-left">{{formatDatetime(rec.easymeta__Next_Billed_At__c)}}</td>
                                <td class="text-left">{{rec.easymeta__Billing_Cycle__c}}</td>
                            </tr>
                        </tbody>
                    </q-markup-table>
                    <q-card-section v-else class="text-center q-py-xl">
                        <div class="q-mb-md">You haven't subscribed Metaforce yet!</div>
                        <q-btn label="Subscribe Now" icon-right="arrow_circle_right" class="text-primary" to="/pricing" no-caps></q-btn>
                    </q-card-section>
                </q-card>
            </div>
        </div>
        <div class="col-1"></div>
    </div>
    <new-case-popup ref="newCasePopupCmp" :customer="customer" @onSubmitted="loadCustomerCases"></new-case-popup>
    <case-detail-popup ref="caseDetailCmp"></case-detail-popup>
</template>

<script>
import NewCasePopup from 'src/components/NewCasePopup.vue';
import CaseDetailPopup from 'src/components/CaseDetailPopup.vue';

import { useCustomerStore } from 'src/stores/customer';
import { mapActions, mapState } from 'pinia'

import { formatDatetime } from 'src/common/utils'
import { notifyError, notifyOk } from 'src/common/notify';

export default {
    components: { NewCasePopup, CaseDetailPopup },
    data () {
        return {
            myCases: [],
            isAccountUpdating: false,
            isCasesLoading: false,

            nameRules: [val => (val && val.length > 0) || 'Required'],
        }
    },
    computed: {
        ...mapState(useCustomerStore, ['loginToken', 'customer', 'subscriptions', 'hasActiveSubscription']),
    },
    methods: {
        ...mapActions(useCustomerStore, ['refreshCustomer', 'getCustomerCases']),
        formatDatetime,
        viewCaseDetail (caseRecord) { this.$refs.caseDetailCmp.show(caseRecord); },
        async updateAccount () {
            let isValid = await this.$refs.formCmp.validate();
            if (isValid) {
                this.isAccountUpdating = true;
                let result = await useCustomerStore().updateCustomerName({
                    firstname: this.customer.easymeta__FirstName__c,
                    lastname: this.customer.easymeta__LastName__c
                });
                if (result.isSuccess) {
                    notifyOk('Updated successfully.');
                } else {
                    notifyError(result.message);
                }
                this.isAccountUpdating = false;
            }
        },
        async loadCustomerCases () {
            this.isCasesLoading = true;
            this.myCases = (await this.getCustomerCases())?.cases || [];
            this.isCasesLoading = false;
        }
    },
    async mounted () {
        if (this.loginToken) {
            await this.loadCustomerCases();
        }

        this.$nextTick(() => {
            let action = this.$route.query?.action;
            if (action == 'newcase') {
                this.$refs.newCasePopupCmp.show()
            }
        })
    }
}
</script>
