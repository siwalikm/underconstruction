<template>
    <div class="flex-center flex-column full-height-vh" :class="{ body_warning: wrongCode, body_success: success }">
        <div class="title">
            {{ title }}
        </div>

        <div v-if="attempts_left != false">
            <p>{{ attempts_left }}</p>
        </div>

        <div v-if="seconds_message != false">
            <p>{{ seconds_message }}</p>
        </div>

        <div class="panel flex flex-column" :class="{ wrong_code: wrongCode, success_code: success }">
            <div class="flex-one">
                <div class="flex full-height">
                    <div class="flex-one number">
                        <div class="flex-center full-height">
                            <h3>{{ code[0] }}</h3>
                        </div>
                    </div>

                    <div class="flex-one number">
                        <div class="flex-center full-height">
                            <h3>{{ code[1] }}</h3>
                        </div>
                    </div>

                    <div class="flex-one number">
                        <div class="flex-center full-height">
                            <h3>{{ code[2] }}</h3>
                        </div>
                    </div>

                    <div class="flex-one number">
                        <div class="flex-center full-height">
                            <h3>{{ code[3] }}</h3>
                        </div>
                    </div>
                </div>
            </div>

            <div class="flex-three">
                <div class="flex flex-column full-height">
                    <div class="flex-one">
                        <div class="flex full-height">
                            <div class="flex-one number" @click="addNumber(7)">
                                <div class="flex-center full-height">
                                    <h3>7</h3>
                                </div>
                            </div>

                            <div class="flex-one number" @click="addNumber(8)">
                                <div class="flex-center full-height">
                                    <h3>8</h3>
                                </div>
                            </div>

                            <div class="flex-one number" @click="addNumber(9)">
                                <div class="flex-center full-height">
                                    <h3>9</h3>
                                </div>
                            </div>
                        </div>
                    </div>

                    <div class="flex-one">
                        <div class="flex full-height">
                            <div class="flex-one number" @click="addNumber(4)">
                                <div class="flex-center full-height">
                                    <h3>4</h3>
                                </div>
                            </div>

                            <div class="flex-one number" @click="addNumber(5)">
                                <div class="flex-center full-height">
                                    <h3>5</h3>
                                </div>
                            </div>

                            <div class="flex-one number" @click="addNumber(6)">
                                <div class="flex-center full-height">
                                    <h3>6</h3>
                                </div>
                            </div>
                        </div>
                    </div>

                    <div class="flex-one">
                        <div class="flex full-height">
                            <div class="flex-one number" @click="addNumber(1)">
                                <div class="flex-center full-height">
                                    <h3>1</h3>
                                </div>
                            </div>

                            <div class="flex-one number" @click="addNumber(2)">
                                <div class="flex-center full-height">
                                    <h3>2</h3>
                                </div>
                            </div>

                            <div class="flex-one number" @click="addNumber(3)">
                                <div class="flex-center full-height">
                                    <h3>3</h3>
                                </div>
                            </div>
                        </div>
                    </div>

                    <div class="flex-one">
                        <div class="flex full-height">
                            <div class="flex-one number" @click="addNumber(0)">
                                <div class="flex-center full-height">
                                    <h3>0</h3>
                                </div>
                            </div>

                            <div class="flex-two number" @click="back()">
                                <div class="flex-center full-height">
                                    <h3>{{ backButton }}</h3>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        props: ['title', 'backButton', 'redirectUrl'],

        data() {
            return {
                code: [],
                position: 0,
                wrongCode: false,
                success: false,
                seconds_message: false,
                attempts_left: false,
                seconds: 0
            }
        },

        mounted() {
            this.registerKeys();
            this.resetCode();
        },

        methods: {

            /**
             * Add entered number to code array.
             */

            addNumber(number) {
                if(!this.seconds_message) {
                    this.setNumber(number);

                    if(this.codeIsComplete()) {
                        axios.post("/under/check", { "code": this.code.join("") })
                            .then(() => {
                                this.success = true;
                                window.location.href = this.redirectUrl;
                            })
                            .catch((error) => {
                                this.wrongCode = true;
                                setTimeout(() => this.wrongCode = false, 5000);

                                if(this.tooManyAttempts(error)) {
                                    this.countDown(error.response.data.seconds_message);
                                }

                                else {
                                    this.attempts_left = error.response.data.attempts_left;
                                }

                                this.resetCode();
                            });
                    }
                }
            },

            /**
             * Extract the seconds out of the string. Then
             * start a timer and decrement it every second.
             */

            countDown(message) {
                this.attempts_left = false;
                this.seconds_message = message;

                let timer = setInterval(() => {
                    if(this.seconds == 1) {
                        this.seconds_message = false;
                        this.seconds = 0;
                        clearInterval(timer);
                    }

                    else {
                        this.seconds_message = this.seconds_message.replace(/\d+/g, (match) => {
                            this.seconds = parseInt(match) - 1;
                            return this.seconds;
                        });
                    }
                }, 1000);
            },

            tooManyAttempts(error) {
                return error.response.data.too_many_attemps;
            },

            back() {
                if(this.position > 0) {
                    Vue.set(this.code, this.position -1, "*");
                    this.position--;
                }
            },

            resetCode() {
                this.code = ['*', '*', '*', '*'];
                this.position = 0;
            },

            /**
             * Set number at the correct array position.
             */

            setNumber(number) {
                Vue.set(this.code, this.position, number);
                this.position++;
            },

            /**
             * Check if all numbers are entered.
             */

            codeIsComplete() {
                return this.position == 4;
            },

            /**
             * Register all keyboard numbers.
             */

            registerKeys() {
                document.addEventListener("keydown", (e) => {
                    switch (e.keyCode) {
                        case 48:
                            this.addNumber(0);
                            break;
                        case 49:
                            this.addNumber(1);
                            break;
                        case 50:
                            this.addNumber(2);
                            break;
                        case 51:
                            this.addNumber(3);
                            break;
                        case 52:
                            this.addNumber(4);
                            break;
                        case 53:
                            this.addNumber(5);
                            break;
                        case 54:
                            this.addNumber(6);
                            break;
                        case 55:
                            this.addNumber(7);
                            break;
                        case 56:
                            this.addNumber(8);
                            break;
                        case 57:
                            this.addNumber(9);
                            break;
                        case 8:
                            this.back();
                            break;
                    }
                });
            }
        }
    }
</script>


<style lang="sass" scoped>

    $success: #27ae60
    $warning: #e74c3c
    $box-shadow: 0 2px 3px 0
    $shadow-color: rgba(0,0,0,.16)
    $mobile-break-point: 750px

    .body_warning
        color: $warning

    .body_success
        color: $success

    .flex
        display: flex

    .flex-one
        flex: 1

    .flex-two
        flex: 1

    .flex-three
        flex: 3

    .full-height-vh
        height: 100vh

    .full-height
        height: 100%

    .flex-center
        @extend .flex
        align-items: center
        justify-content: center

    .flex-column
        flex-direction: column

    .panel
        width: 300px
        height: 400px
        background: #F8F8FA
        box-shadow: $box-shadow $shadow-color
        border-radius: 6px

    .wrong_code
        box-shadow: $box-shadow $warning

    .success_code
        box-shadow: $box-shadow $success

    .number
        margin: 10px
        border-bottom: 1px solid #DCDCDE
        cursor: pointer
        div
            h3
                font-size: 15px
                font-weight: 900
        &:hover
            box-shadow: $box-shadow $shadow-color

    .title
        font-size: 84px
        margin-bottom: 40px

    @media only screen and (max-width: $mobile-break-point)
        .title
            display: none
</style>