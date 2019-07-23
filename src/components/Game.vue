<template>
    <div class="game">
        <span class="status">{{ statusLabel }}</span>

        <grid ref="grid" :can-play="status === 1" @on-defeat="onDefeat()" @on-victory="onVictory()"></grid>

        <span class="score">Score : {{ score }}</span>

        <button @click="start()">Réinitialiser la partie</button>
    </div>
</template>

<script>
    import Grid from "./Grid";

    const STATUS_IN_PROGRESS = 1;
    const STATUS_VICTORY = 2;
    const STATUS_DEFEAT = 3;

    export default {
        name: "Game",
        components: {
            Grid,
        },
        data() {
            return {
                status: null,
                score: 0,
            };
        },
        mounted() {
            this.start();
        },
        methods: {
            start() {
                this.status = STATUS_IN_PROGRESS;
                this.$refs.grid.init();
            },
            onDefeat() {
                this.status = STATUS_DEFEAT;
            },
            onVictory() {
                this.status = STATUS_VICTORY;
            },
        },
        computed: {
            statusLabel() {
                switch (this.status) {
                    case STATUS_IN_PROGRESS:
                        return 'En cours...';
                    case STATUS_DEFEAT:
                        return 'Défaite !';
                    case STATUS_VICTORY:
                        return 'Victoire !';
                }
            },
        },
    }
</script>

<style lang="scss" scoped>
    .game {
        font-family: "Helvetica Neue", serif;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
        margin-top: 50px;

        .score, .status {
            margin-bottom: 15px;
        }
    }
</style>
