<!-- =========================================================================================
  File Name: DataListListView.vue
  Description: Data List - List View
  ----------------------------------------------------------------------------------------
  Item Name: Vuexy - Vuejs, HTML & Laravel Admin Dashboard Template
  Author: Pixinvent
  Author URL: http://www.themeforest.net/user/pixinvent
========================================================================================== -->

<template>
    <div id="data-list-list-view" class="data-list-container">
        <!-- <data-view-sidebar
      :isSidebarActive="addNewDataSidebar"
      @closeSidebar="toggleDataSidebar"
      :data="sidebarData"
        />-->
        <data-view-sidebar :isSidebarActive="addNewDataSidebar" @closeSidebar="toggleDataSidebar" :data="sidebarData" />

        <vs-table ref="table" multiple v-model="selected" search pagination :max-items="itemsPerPage" :data="recipesList" style="z-index = -1;">
            <div slot="header" class="flex flex-wrap-reverse items-center flex-grow justify-between">
                <div class="flex flex-wrap-reverse items-center data-list-btn-container">
                    <div
                        class="btn-add-new p-3 mb-4 mr-4 rounded-lg cursor-pointer flex items-center justify-center text-lg font-medium text-base text-primary border border-solid border-primary"
                        @click="addNewData"
                    >
                        <feather-icon icon="PlusIcon" svgClasses="h-4 w-4" />
                        <span class="ml-2 text-base text-primary">Thêm mới</span>
                    </div>
                    <div
                        class="btn-add-new p-3 mb-4 mr-4 rounded-lg cursor-pointer flex items-center justify-center text-lg font-medium text-base text-primary border border-solid border-primary"
                        @click="reloadData"
                    >
                        <feather-icon icon="RefreshCcwIcon" svgClasses="h-4 w-4" />
                        <span class="ml-2 text-base text-primary">Reload</span>
                    </div>
                </div>

                <vs-dropdown vs-trigger-click class="cursor-pointer mb-4 mr-4 items-per-page-handler">
                    <div class="p-4 border border-solid d-theme-border-grey-light rounded-full d-theme-dark-bg cursor-pointer flex items-center justify-between font-medium">
                        <span class="mr-2">
                            {{ currentPage * itemsPerPage - (itemsPerPage - 1) }} -
                            {{ recipesList.length - currentPage * itemsPerPage > 0 ? currentPage * itemsPerPage : recipesList.length }} of {{ queriedItems }}
                        </span>
                        <feather-icon icon="ChevronDownIcon" svgClasses="h-4 w-4" />
                    </div>
                    <vs-dropdown-menu>
                        <vs-dropdown-item @click="itemsPerPage = 4">
                            <span>4</span>
                        </vs-dropdown-item>
                        <vs-dropdown-item @click="itemsPerPage = 10">
                            <span>10</span>
                        </vs-dropdown-item>
                        <vs-dropdown-item @click="itemsPerPage = 15">
                            <span>15</span>
                        </vs-dropdown-item>
                        <vs-dropdown-item @click="itemsPerPage = 20">
                            <span>20</span>
                        </vs-dropdown-item>
                    </vs-dropdown-menu>
                </vs-dropdown>
            </div>

            <template slot="thead">
                <vs-th sort-key="id">id</vs-th>
                <vs-th sort-key="title">Tên công thức</vs-th>
                <vs-th sort-key="dish_types">Loại</vs-th>
                <vs-th sort-key="servings">Khẩu phần</vs-th>
                <vs-th sort-key="readyInMinutes">Thời lượng</vs-th>
                <vs-th sort-key="step">Số bước thực hiện</vs-th>
                <vs-th sort-key="summary">Thới thiệu</vs-th>
                <vs-th>Chức năng</vs-th>
            </template>

            <template slot-scope="{ data }">
                <tbody>
                    <vs-tr :data="tr" :key="indextr" v-for="(tr, indextr) in data">
                        <vs-td>
                            <p class="product-category">{{ tr.id }}</p>
                        </vs-td>

                        <vs-td>
                            <p class="product-name font-medium truncate">{{ tr.title }}</p>
                        </vs-td>

                        <vs-td>
                            <p class="product-category">{{ tr.dish_types }}</p>
                        </vs-td>

                        <vs-td>
                            <p class="product-category">{{ tr.servings }}</p>
                        </vs-td>

                        <vs-td>
                            <p class="product-category">{{ tr.readyInMinutes }} phút</p>
                        </vs-td>

                        <vs-td>
                            <p class="product-category">{{ tr.inductions.length }} bước</p>
                        </vs-td>

                        <vs-td>
                            <p class="product-name font-medium truncate">{{ tr.summary }}</p>
                        </vs-td>

                        <vs-td class="whitespace-no-wrap">
                            <feather-icon icon="EditIcon" svgClasses="w-5 h-5 hover:text-primary stroke-current" @click.stop="editData(tr)" />
                            <feather-icon icon="TrashIcon" svgClasses="w-5 h-5 hover:text-danger stroke-current" class="ml-2" />
                        </vs-td>
                    </vs-tr>
                </tbody>
            </template>
        </vs-table>
    </div>
</template>

<script>
import DataViewSidebar from './RecipesDetailSideView';

export default {
    components: {
        DataViewSidebar
    },
    data() {
        return {
            selected: [],
            itemsPerPage: 4,
            isMounted: false,

            recipesList: [],

            addNewDataSidebar: false,
            sidebarData: {}
        };
    },
    computed: {
        currentPage() {
            if (this.isMounted) {
                return this.$refs.table.currentx;
            }
            return 0;
        },
        queriedItems() {
            return this.$refs.table ? this.$refs.table.queriedResults.length : this.recipesList.length;
        }
    },
    methods: {
        reloadData() {
            this.getList();
        },
        addNewData() {
            console.log('add click');
            this.sidebarData = {};
            this.toggleDataSidebar({ val: true, isSubmit: undefined });
        },
        editData(data) {
            console.log('edit data ' + JSON.stringify(data));
            this.sidebarData = data;
            this.toggleDataSidebar({ val: true, isSubmit: undefined });
        },
        async getList() {
            this.$vs.loading();

            this.recipesList = [];

            await this.$store
                .dispatch('recipesManager/fetchRecipes')
                .then((result) => {
                    console.log(result);
                    this.recipesList = result.data;
                })
                .catch((err) => {
                    console.log(err);

                    this.$vs.notify({
                        color: 'danger',
                        title: 'Error',
                        position: 'top-center',
                        text: err.message + '. Please try again!'
                    });
                });

            console.log(this.recipesList);

            this.$vs.loading.close();
        },
        async toggleDataSidebar(datasibe = { val: false, isSubmit: undefined }) {
            console.log('toggleDataSidebar has been call');
            this.addNewDataSidebar = datasibe.val;
            if (datasibe.isSubmit) {
                console.log('reload when submit');
                await this.getList();
            }
        }
    },
    created() {},
    async mounted() {
        this.isMounted = true;
        await this.getList();
    }
};
</script>

<style lang="scss">
#data-list-list-view {
    .vs-con-table {
        /*
      Below media-queries is fix for responsiveness of action buttons
      Note: If you change action buttons or layout of this page, Please remove below style
    */
        @media (max-width: 689px) {
            .vs-table--search {
                margin-left: 0;
                max-width: unset;
                width: 100%;

                .vs-table--search-input {
                    width: 100%;
                }
            }
        }

        @media (max-width: 461px) {
            .items-per-page-handler {
                display: none;
            }
        }

        @media (max-width: 341px) {
            .data-list-btn-container {
                width: 100%;

                .dd-actions,
                .btn-add-new {
                    width: 100%;
                    margin-right: 0 !important;
                }
            }
        }

        .product-name {
            max-width: 23rem;
        }

        .vs-table--header {
            display: flex;
            flex-wrap: wrap;
            margin-left: 1.5rem;
            margin-right: 1.5rem;
            > span {
                display: flex;
                flex-grow: 1;
            }

            .vs-table--search {
                padding-top: 0;

                .vs-table--search-input {
                    padding: 0.9rem 2.5rem;
                    font-size: 1rem;

                    & + i {
                        left: 1rem;
                    }

                    &:focus + i {
                        left: 1rem;
                    }
                }
            }
        }

        .vs-table {
            border-collapse: separate;
            border-spacing: 0 1.3rem;
            padding: 0 1rem;

            tr {
                box-shadow: 0 4px 20px 0 rgba(0, 0, 0, 0.05);
                td {
                    padding: 20px;
                    &:first-child {
                        border-top-left-radius: 0.5rem;
                        border-bottom-left-radius: 0.5rem;
                    }
                    &:last-child {
                        border-top-right-radius: 0.5rem;
                        border-bottom-right-radius: 0.5rem;
                    }
                }
                td.td-check {
                    padding: 20px !important;
                }
            }
        }

        .vs-table--thead {
            th {
                padding-top: 0;
                padding-bottom: 0;

                .vs-table-text {
                    text-transform: uppercase;
                    font-weight: 600;
                }
            }
            th.td-check {
                padding: 0 15px !important;
            }
            tr {
                background: none;
                box-shadow: none;
            }
        }

        .vs-table--pagination {
            justify-content: center;
        }
    }
}
</style>
