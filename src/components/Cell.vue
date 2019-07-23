<template>
    <div class="cell" :class="status" @click.left.stop.prevent="dig()" @click.right.stop.prevent="flag()">
        {{ status === 'dug' && cell.minesCount > 0 ? cell.minesCount : icons[status] }}
    </div>
</template>

<script>
    export default {
        name: "Cell",
        props: {
            cell: {
                type: Object,
                required: true,
            },
            canPlay: {
                type: Boolean,
                default: true,
            },
        },
        data() {
            return {
                icons: {
                    untouched: '',
                    dug: '',
                    flagged: '🚩',
                    detonated: '💥',
                },
            };
        },
        methods: {
            dig(propagation = true) {
                if (!this.canPlay || this.cell.dug) {
                    return;
                }

                this.cell.flagged = false;
                this.cell.dug = true;

                if (propagation) {
                    this.cell.adjCells.forEach((adjCell) => {
                        if (!adjCell.withBomb && !adjCell.dug) {
                            this.$parent.$refs['cell_' + adjCell.index][0].dig(adjCell.minesCount === 0);
                        }
                    });
                }

                console.log('[' + this.cell.index + ' - DIG] ' + this.cell.dug);

                this.$emit('on-dig-cell');
            },
            flag() {
                if (!this.canPlay || this.cell.dug) {
                    return;
                }

                this.cell.flagged = !this.cell.flagged;

                console.log('[' + this.cell.index + ' - FLAG] ' + this.cell.flagged);
            },
        },
        computed: {
            detonated() {
                return this.cell.withBomb && this.cell.dug;
            },
            status() {
                if (this.detonated) {
                    return 'detonated';
                }

                if (this.cell.dug) {
                    return 'dug';
                }

                if (this.cell.flagged) {
                    return 'flagged';
                }

                if (this.cell.withBomb) {
                    return 'bombed';
                }

                return 'untouched';
            },
        },
    }
</script>

<style lang="scss" scoped>
    .cell {
        background-color: beige;
        border: 1px solid black;
        height: 38px;
        width: 38px;
        text-align: center;
        line-height: 38px;
        cursor: pointer;

        &.bombed {
            background-color: darkcyan;
        }

        &.detonated {
            background-color: red;
        }

        &.flagged {
            background-color: lightgrey;
        }

        &.dug {
            background-color: white;
        }
    }
</style>
