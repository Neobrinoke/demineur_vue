<template>
    <div class="grid" :style="style">
        <cell v-for="cell in cells" :can-play="canPlay" :cell="cell" :ref="'cell_' + cell.index" :key="cell.index" @on-dig-cell="onDigCell()"></cell>
    </div>
</template>

<script>
    import Cell from "./Cell";
    import Config from "../config";

    export default {
        name: "Grid",
        props: {
            canPlay: {
                type: Boolean,
                default: true,
            },
        },
        components: {
            Cell,
        },
        data() {
            return {
                cells: [],
            };
        },
        methods: {
            init() {
                this.cells = [];

                const length = Config.rows * Config.columns;

                for (let i = 0; i < length; i++) {
                    this.cells.push({
                        index: i,
                        withBomb: Config.minesCount > i,
                        flagged: false,
                        dug: false,
                        minesCount: 0,
                        adjCells: [],
                    });
                }

                let index = -1;
                while (++index < length) {
                    const rand = index + Math.floor(Math.random() * (length - index));
                    let cell = this.cells[rand];

                    this.cells[rand] = this.cells[index];
                    this.cells[index] = cell;

                    this.cells[rand].index = rand;
                    this.cells[index].index = index;
                }

                let topIds = [];
                for (let topId = 0; topIds.length !== Config.columns; topId++) {
                    topIds.push(topId);
                }

                let leftIds = [];
                for (let leftId = 0; leftIds.length !== Config.rows; leftId += Config.columns) {
                    leftIds.push(leftId);
                }

                let rightIds = [];
                for (let rightId = Config.columns - 1; rightIds.length !== Config.rows; rightId += Config.columns) {
                    rightIds.push(rightId);
                }

                let bottomIds = [];
                for (let bottomId = length - 1; bottomIds.length !== Config.columns; bottomId--) {
                    bottomIds.push(bottomId);
                }

                this.cells = this.cells.map((cell) => {
                    if (!cell.withBomb) {
                        if (!topIds.includes(cell.index) && this.cells[cell.index - Config.columns]) {
                            cell.adjCells.push(this.cells[cell.index - Config.columns]);
                        }

                        if (!leftIds.includes(cell.index) && this.cells[cell.index - 1]) {
                            cell.adjCells.push(this.cells[cell.index - 1]);
                        }

                        if (!rightIds.includes(cell.index) && this.cells[cell.index + 1]) {
                            cell.adjCells.push(this.cells[cell.index + 1]);
                        }

                        if (!bottomIds.includes(cell.index) && this.cells[cell.index + Config.columns]) {
                            cell.adjCells.push(this.cells[cell.index + Config.columns]);
                        }

                        cell.adjCells.forEach((adjCell) => {
                            if (adjCell.withBomb) {
                                cell.minesCount += 1;
                            }
                        });
                    }

                    return cell;
                });

                this.cells.forEach((cell) => {
                    console.log('[' + cell.index + ' - ' + cell.minesCount + '] => [' + cell.adjCells.map(adjCell => adjCell.index) + ']')
                });
            },
            onDigCell() {
                if (this.defeat) {
                    this.$emit('on-defeat');
                }

                if (this.victory) {
                    this.$emit('on-victory');
                }
            },
        },
        computed: {
            style() {
                return {
                    width: 40 * Config.columns + 'px',
                };
            },
            victory() {
                for (let cell of this.cells) {
                    if ((!cell.dug && !cell.withBomb) || (cell.withBomb && cell.dug)) {
                        return false;
                    }
                }

                return true;
            },
            defeat() {
                for (let cell of this.cells) {
                    if (cell.withBomb && cell.dug) {
                        return true;
                    }
                }

                return false;
            },
        },
    }
</script>

<style lang="scss" scoped>
    .grid {
        display: flex;
        box-sizing: content-box;
        flex-wrap: wrap;
        margin-bottom: 20px;
    }
</style>
