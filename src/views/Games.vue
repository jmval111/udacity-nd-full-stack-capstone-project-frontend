<template>
    <v-container fluid>
        <!--        <v-dialog v-model="isGameSaveDialogOpen"></v-dialog>-->
        <games-save-game-dialog
            v-model="isGameSaveDialogOpen"
            :game="openedGame"
            @save:game="gameSaved"
        ></games-save-game-dialog>

        <v-layout column>
            <v-flex filter-row>
                <v-layout>
                    <v-text-field
                        v-model="gameSearchTerm"
                        single-line
                        hide-details
                        class="filter-row--search-term"
                    ></v-text-field>
                    <v-spacer></v-spacer>
                    <v-btn v-if="can('create:game')" @click="openSaveGameDialog">{{ $t('create') }}</v-btn>
                </v-layout>
            </v-flex>
            <v-flex>
                <v-data-table
                    :headers="gameHeader"
                    :items="gameList"
                    item-key="id"
                    disable-sort
                    :page="gamePage"
                    :items-per-page.sync="gamePerPage"
                    :footer-props.sync="gamePageFooterProps"
                    :server-items-length="gameCount"
                    :loading="gameListLoading"
                    class="v-data-table--full-page"
                >
                    <template v-slot:item.action="{ item }">
                        <v-icon
                            v-if="can('edit:game')"
                            small
                            @click="openSaveGameDialog(item)"
                        >
                            create
                        </v-icon>
                    </template>
                </v-data-table>
            </v-flex>
        </v-layout>
    </v-container>
</template>

<script>
    import AuthMixin from '@/mixins/AuthMixin';
    import GamesSaveGameDialog from "@/views/Games/GamesSaveGameDialog";
    import GamesApi from '@/api/games';

    export default {
        name: "Games",
        mixins: [AuthMixin],
        components: {GamesSaveGameDialog},
        data() {
            return {
                gameList: [],
                gameCount: 0,
                gamePerPage: 20,
                gamePage: 1,
                gamePageFooterProps: {
                    itemsPerPageOptions: [20, 30, 50],
                    showCurrentPage: true,
                    showFirstLastPage: true,
                },
                gameHeader: [
                    {text: this.$tc('name', 1), value: 'name'},
                    {text: this.$tc('action', 1), value: 'action', sortable: false},
                    // {text: 'name', value: 'name'},
                ],
                gameListLoading: false,
                gameListLoadingCount: 0,
                gameSearchTerm: '',
                filterCount: 0,
                isGameSaveDialogOpen: false,
                openedGame: null,
            }
        },
        computed: {},
        watch: {
            gamePerPage() {
                this.loadGames(true)
            },
            gamePage() {
                this.loadGames();
            },
            gameSearchTerm() {
                this.filterGames();
            },
        },
        created() {
            window.a = this;
            this.loadGames();
        },
        methods: {
            async loadGames(reset) {
                if (reset) {
                    this.gameList = [];
                }
                this.gameListLoading = true;
                let loadCount = ++this.gameListLoadingCount;
                let filter = {};
                if (this.gameSearchTerm) filter.searchTerm = this.gameSearchTerm;
                await GamesApi.requests.getGames({}, filter, this.gamePage, this.gamePerPage)
                    .then(response => {
                        if (loadCount === this.gameListLoadingCount) {
                            this.gameList = response.data.games;
                            this.gameCount = response.data.total_games;
                            this.gameListLoading = false;
                        }
                    })
                    .catch(response => {
                        // TODO: Show snackbar error
                    });
                if (loadCount === this.gameListLoadingCount) {
                    this.gameListLoading = false;
                }
            },
            filterGames() {
                let filterCount = ++this.filterCount;
                setTimeout(() => {
                    if (filterCount === this.filterCount)
                        this.loadGames(true);
                }, 300);
            },
            openSaveGameDialog(game) {
                this.openedGame = game;
                this.isGameSaveDialogOpen = true;
            },
            gameSaved(game) {
                this.loadGames(true);
                this.closeSaveGameDialog();
            },
            closeSaveGameDialog() {
                this.isGameSaveDialogOpen = false;
            }
        },
    }
</script>

<style scoped>

</style>