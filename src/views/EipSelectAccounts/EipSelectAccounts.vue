<template>
    <div class="view-settings">
        <h1>Select accounts to use</h1>
        <p>For: {{ site }}</p>
        <div class="settings-form">
            <f-card class="f-card-double-padding">
                <f-form @f-form-submit="onSubmit">
                    <fieldset>
                        <div v-for="account in accounts" :key="account">
                            <f-toggle-button
                                :name="account.address"
                                :label="account.name"
                                :checked="account.sites && account.sites.includes(site)"
                            />
                            <account-name :account="account" hide-name="true" />
                        </div>

                        <div class="align-center form-buttons">
                            <button type="submit" class="btn large break-word" style="max-width: 100%;">
                                Select
                            </button>
                        </div>
                    </fieldset>
                </f-form>
            </f-card>
        </div>
    </div>
</template>

<script>
import { mapGetters } from 'vuex';
import FToggleButton from '@/components/core/FToggleButton/FToggleButton';
import FForm from '@/components/core/FForm/FForm';
import FCard from '@/components/core/FCard/FCard';
import { SET_ACCOUNT } from '@/store/mutations.type';
import AccountName from '@/components/AccountName/AccountName';

export default {
    name: 'EipSelectAccounts',

    components: { FForm, FCard, FToggleButton, AccountName },

    computed: {
        ...mapGetters(['currentAccount', 'accounts']),
        site: function () {
            return this.$route.params.site;
        },
    },

    methods: {
        onSubmit(_event) {
            const { detail } = _event;
            let site = this.site;
            let selectedAccounts = Object.keys(detail.data);

            this.accounts.forEach((account, index) => {
                let wasSelected = account.sites && account.sites.includes(site);
                let isSelected = selectedAccounts.includes(account.address);
                if (isSelected && !wasSelected) {
                    let sites = account.sites || [];
                    sites.push(site);
                    this.$store.commit(SET_ACCOUNT, { ...account, sites, index });
                }
                if (!isSelected && wasSelected) {
                    let sites = account.sites.filter((value) => value !== site);
                    this.$store.commit(SET_ACCOUNT, { ...account, sites, index });
                }
            });
            window.close();
        },
    },
};
</script>
