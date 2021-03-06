<template>
    <v-row class="px-lg-4">
        <v-col cols="12">
            <v-btn
            fab
            small
            :to="{ name: 'MenuListView'}"
            class="mr-2 text-decoration-none"><v-icon>mdi-chevron-left</v-icon></v-btn>
            <v-btn
                class="mr-2"
                @click="preview">
                Preview Menu
            </v-btn>
            <v-btn
                @click="generateQr">
                Generate QR Code
            </v-btn>
        </v-col>

        <v-col cols="12">
            <v-btn
                color="deep-purple darken-2 white--text"
                class="d-block"
                :disabled="isCreate"
                @click="overlay.category = true"
            >Manage Category</v-btn>
            <div class="p-1 shadow" style="border-left: 4px solid #512DA8; border-bottom: 4px solid #512DA8">
                <MenuCategory
                    :categories="menu.categories"
                    :isCreate="isCreate"
                    ref="MenuCategory"
                    @changeList="changeList">
                </MenuCategory>
            </div>
        </v-col>

        <v-col cols="12">
            <v-btn
                block
                color="success"
                @click="createItem"
                :disabled="isCreate||!category"
            >Create Item</v-btn>
            <center v-if="!category"><p class="font-weight-bold">Add at least one category in "Manage Category"</p></center>
            <ItemDetail
                v-if="newItem"
                :item="newItem"
                :isCreate="isCreate"
                :categories="menu.categories"
                @refreshData="refreshData"
                @cancelCreate="cancelCreate"
            />
        </v-col>
        <v-col cols="12">
            <ItemList :items="itemList" :categories="menu.categories" @refreshData="refreshData" />
        </v-col>
        <v-overlay :value="overlay.category" :dark="false">
            <CategoryManager
                :categories="menu.categories"
                :menuId="menu._id"
                @closeOverlay="overlay.category = false"
                @refreshData="refreshData"
                ></CategoryManager>
        </v-overlay>
        <v-overlay :value="overlay.qrcode">
            <v-row justify="center" alignContent="center" class="qrcode-wrapper">
                <v-col cols="10" lg="12" class="d-flex justify-center">
                    <img :src="qrcode" class="qrcode-img"/>
                </v-col>
                <v-col cols="10" lg="12" class="d-flex justify-center p-0">
                    <v-btn color="success" @click="download" width="45%" class="mx-1 mb-2 mx--lg-2">Download</v-btn>
                    <v-btn @click="overlay.qrcode = false" width="45%" class="mx-1 mb-2 mx--lg-2">close</v-btn>
                </v-col>
            </v-row>
        </v-overlay>
    </v-row>
</template>

<script>
import ItemList from './ItemList.vue';
import ItemDetail from './ItemDetail.vue';
import CategoryManager from './CategoryManager.vue';
import MenuCategory from '@/app/components/menu/MenuCategory.vue';
import axios from 'axios';
import { path } from '@/constant.js';
import QRCode from 'qrcode';
import AxiosHandlerMixin from '@/app/mixins/axiosHandler.mixin.js';

export default {
    mixins: [AxiosHandlerMixin],
    components: {
        ItemList,
        ItemDetail,
        CategoryManager,
        MenuCategory
    },
    data() {
        return {
            menu: { _id: '' },
            itemList: [],
            category: null,
            newItem: null,
            isCreate: false,
            qrcode: '',
            overlay: {
                category: false,
                qrcode: false
            }
        };
    },
    created() {
        axios
            .get(path.menus.index + `/${this.$route.params.id}`)
            .then((response) => {
                this.menu = response.data;
                this.changeList();
            })
            .catch((error) => {
                this.axiosErrorHandler(error);
            });
    },
    methods: {
        refreshData(category, isCreate = false) {
            axios
                .get(path.menus.index + `/${this.$route.params.id}`)
                .then((response) => {
                    this.menu = response.data;

                    if (isCreate) {
                        this.cancelCreate();
                    }

                    if (this.overlay.category) {
                        if (!this.menu.categories[0]) {
                            this.itemList = [];
                            this.category = null;
                        }
                        this.overlay.category = false;
                    }

                    this.changeList(category);
                })
                .catch((error) => {
                    this.axiosErrorHandler(error);
                });
        },
        changeList(category = null) {
            if (category) {
                this.category = category;
            } else {
                this.category = this.menu.categories[0];
            }

            if (this.category) {
                this.itemList = this.menu.sortedItems[this.category];
            }
        },
        createItem() {
            if (!this.isCreate) {
                this.isCreate = true;
                this.newItem = {
                    code: '',
                    name: '',
                    category: this.category,
                    price: 0.0,
                    img: '/placeholder.png',
                    menuId: this.menu._id
                };
            }
        },
        cancelCreate() {
            this.isCreate = false;
            this.newItem = null;
        },
        preview() {
            const route = this.$router.resolve({
                name: 'MenuViewReadonly',
                params: { id: this.menu._id }
            });
            window.open(route.href);
        },
        generateQr() {
            this.overlay.qrcode = true;

            const route = this.$router.resolve({
                name: 'MenuViewReadonly',
                params: { id: this.menu._id }
            });

            QRCode.toDataURL(
                `${window.location.origin}/${route.href}`,
                { errorCorrectionLevel: 'H', width: '500' },
                (er, url) => {
                    this.qrcode = url;
                }
            );
        },
        download() {
            const link = document.createElement('a');
            link.download = 'qrcode.png';
            link.href = this.qrcode;
            link.click();
        },
        goBack() {
            this.$router.push({ name: 'MenuViewEditable' });
        }
    }
};
</script>

<style scoped>
.qrcode-wrapper {
    background: white;
}

.qrcode-img {
    max-height: 100%;
    max-width: 100%;
}

@media screen and (max-width: 959px) {
    .qrcode-wrapper {
        width: 80vw;
    }
}

@media screen and (min-width: 960px) {
    .qrcode-wrapper {
        width: 40vw;
    }
}
</style>
