<template>
    <v-container fluid>
        <v-row>
            <v-col>
                <v-btn color="error lighten-1" class="float-right" v-on:click="cancelAdd">Cancel Wallet Creation</v-btn>
            </v-col>
        </v-row>

        
        <v-stepper
            v-model="e6"
            vertical
        >
            <!-- Step 1 -->
            <v-stepper-step :complete="e6 > 1" step="1">
                Create / Import
            </v-stepper-step>

            <v-stepper-content step="1">
                <v-card flat>
                    <v-row>
                        <v-col lg="6" md="12">
                           <v-sheet outlined color="primary" rounded>
                                <v-card class="pa-2" outlined>
                                    <v-card-title>Create a Wallet</v-card-title>
                                    <v-card-subtitle>I do not already have a wallet.</v-card-subtitle>
                                    <v-card-actions>
                                        <v-btn color="primary" @click="getNewMnemonic">
                                            Create Now
                                        </v-btn>
                                    </v-card-actions>
                                </v-card>
                            </v-sheet>
                        </v-col>
                        <v-col lg="6" md="12">
                           <v-sheet outlined color="secondary" rounded>
                                <v-card class="pa-2" outlined>
                                    <v-card-title>Import a Wallet</v-card-title>
                                    <v-card-subtitle>I already have a wallet. I have my mnemonic words.</v-card-subtitle>
                                    <v-card-actions>
                                        <v-btn color="secondary" @click="importWords">
                                            Import Now
                                        </v-btn>
                                    </v-card-actions>
                                </v-card>
                            </v-sheet>
                        </v-col>
                    </v-row>
                </v-card>
            </v-stepper-content>

            <!-- Step 2 -->
            <v-stepper-step :complete="e6 > 2" step="2">
                Generate Mnemonics
            </v-stepper-step>

            <v-stepper-content step="2">
                <v-card flat>
                    <v-row>
                        <v-col sm="12">
                           <v-sheet outlined color="primary" rounded>
                                <v-card class="pa-2" outlined>
                                    <v-card-title>Record Your Mnemonic Words</v-card-title>
                                    <v-card-subtitle>After this step, Perdix will not be able to show you these words. Please be responsible and write these down. <br/>You will be required to re-enter this words during the next step.</v-card-subtitle>
                                    <v-card-text>
                                        <v-sheet class="pa-5" color="primary lighten-4" rounded="">
                                            <h3>{{newMnemonic}}</h3>
                                        </v-sheet>
                                    </v-card-text>
                                    <v-card-actions>
                                        <v-btn color="primary" @click="importWords">
                                            Continue
                                        </v-btn>
                                    </v-card-actions>
                                </v-card>
                            </v-sheet>
                        </v-col>
                    </v-row>
                </v-card>
            </v-stepper-content>

            <!-- Step 3 -->
            <v-stepper-step :complete="e6 > 3" step="3">
                Add Wallet
            </v-stepper-step>

            <v-stepper-content step="3">
                <v-card flat>
                    <v-row>
                        <v-col sm="12">
                           <v-sheet outlined color="primary" rounded>
                                <v-card class="pa-2" outlined>
                                    <v-card-title>Complete Form</v-card-title>
                                    <v-card-subtitle>Please fill out the form to complete your wallet creation.</v-card-subtitle>
                                     <v-form
                                        v-model="walletFormValid"
                                        ref="form"
                                        lazy-validation
                                    >
                                        <v-card-text>
                                            <v-text-field
                                                v-model="walletForm.name"
                                                :counter="50"
                                                :rules="rules.name"
                                                label="Name"
                                                required
                                                >
                                            </v-text-field>

                                            <v-text-field
                                                v-model="walletForm.mnemonic"
                                                :rules="rules.mnemonic"
                                                label="Mnemonic"
                                                required
                                                >
                                            </v-text-field>

                                            <v-text-field
                                                :append-icon="showPassphrase ? 'mdi-eye' : 'mdi-eye-off'"
                                                v-model="walletForm.passphrase"
                                                :type="showPassphrase ? 'text' : 'password'"
                                                :rules="rules.passphrase"
                                                label="Passphrase"
                                                required
                                                @click:append="showPassphrase = !showPassphrase"
                                                >
                                            </v-text-field>
                                        </v-card-text>
                                        <v-card-actions>
                                            <v-btn color="primary" @click="submitAddWalletForm" :disabled="!walletFormValid">
                                                Submit
                                            </v-btn>
                                        </v-card-actions>
                                    </v-form>
                                </v-card>
                            </v-sheet>
                        </v-col>
                    </v-row>
                </v-card>
            </v-stepper-content>

            
        </v-stepper>
    </v-container>
</template>

<script>
    const { ipcRenderer } = require('electron')
    export default {
        name: "AddWallet",
        data: () => ({
            e6: 1,
            newMnemonic: '',
            walletFormValid: true,
            showPassphrase: false,
            walletForm: {
                name: '',
                mnemonic: '',
                passphrase: ''
            },
            rules: {
                name:[
                    v => !!v || 'Name is required',
                    v => (v && v.length <= 50) || 'Name must be less than 50 characters',
                ],
                mnemonic:[
                    v => !!v || 'Mnemonic is required',
                ],
                passphrase:[
                    v => !!v || 'Passphrase is required',
                    v => (v && v.length >= 10) || 'Passphrase must be greater than 10 characters',
                ]
            }
        }),
        mounted() {
            ipcRenderer.on('res:generate-recovery-phrase', (_, args) => {
                console.log('phrase',args);
                if(!args.error) {
                    this.newMnemonic = args.phrase;
                    this.e6 = 2;
                }else {
                    //err
                }
            })

            ipcRenderer.on('res:add-wallet', (_, args) => {
                console.log('new wallet', args);
                if(!args.error) {
                    this.e6 = 1;
                    this.newMnemonic = '';
                    this.$emit('added-wallet', { wallet: args.wallet });
                }else {
                    //err
                }
            })
        },
        methods: {
            getNewMnemonic: function() {
                ipcRenderer.send('req:generate-recovery-phrase');
            },
            importWords: function() {
                this.newMnemonic = '';
                this.e6 = 3;
            },
            cancelAdd: function(){
                this.e6 = 1;
                this.newMnemonic = '';
                this.$emit('cancel-add');
            },
            submitAddWalletForm: function() {
                ipcRenderer.send('req:add-wallet', this.walletForm);
            }
        }
    }
</script>