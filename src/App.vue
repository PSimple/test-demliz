<template>
    <div id="app">
        <div class="flat-form-wrapper">
            <form id="addForm" class="default-form flat-form" v-on:submit.prevent="addNewRow()">
                <p>
                    <label>{{columns[1]}}</label>
                    <input v-model="newData.fio" name="addFormFio" type="text">
                </p>
                <p>
                    <label>{{columns[2]}}</label>
                    <input v-model="newData.email" name="addFormEmail" type="text">
                </p>
                <p>
                    <label>{{columns[3]}}</label>
                    <input v-model="newData.phone" name="addFormPhone" type="text">
                </p>
                <p>
                    <input type="submit" value="Добавить">
                </p>
            </form>
            <ul class="errors">
                <li v-show="!validation.fio">Поле ФИО не может быть пустым.</li>
                <li v-show="!validation.email">Введите корректный e-mail.</li>
                <li v-show="!validation.phone">Поле Телефон не может быть пустым.</li>
            </ul>
        </div>
        <table>
            <thead>
            <tr>
                <th v-for="key in columns">
                    {{ key }}
                </th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="(item, index) in items">
                <td>{{item.id}}</td>
                <td>{{item.fio}}</td>
                <td>{{item.email}}</td>
                <td>{{item.phone}}</td>
                <td><input type="button" value="Изменить" v-on:click="showModal(index)"/></td>
            </tr>
            </tbody>
        </table>
        <modal v-if="showModalF" @close="closeModal()" @remove="removeRow()" @update="updateRow()">
            <div slot="body">
                <form id="modalForm" class="default-form modal-form">
                    <p>
                        <label>{{columns[0]}}</label>
                        <input name="modalFormId" type="hidden" v-model="currentData.id" v-bind:readonly="true">
                        <span>{{currentData.id}}</span>
                    </p>
                    <p>
                        <label>{{columns[1]}}</label>
                        <input name="modalFormFio" type="text" v-model="currentData.fio">
                    </p>
                    <p>
                        <label>{{columns[2]}}</label>
                        <input name="modalFormEmail" type="text" v-model="currentData.email">
                    </p>
                    <p>
                        <label>{{columns[3]}}</label>
                        <input name="modalFormPhone" type="text" v-model="currentData.phone">
                    </p>
                </form>
                <ul class="errors">
                    <li v-show="!validation.fio">Поле ФИО не может быть пустым.</li>
                    <li v-show="!validation.email">Введите корректный e-mail.</li>
                    <li v-show="!validation.phone">Поле Телефон не может быть пустым.</li>
                </ul>
            </div>
        </modal>
    </div>
</template>

<script>
    var emailRE = /^(([^<>()[\]\\.,;:\s@\"]+(\.[^<>()[\]\\.,;:\s@\"]+)*)|(\".+\"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;

    import appData from '../appdata.json';

    export default {
        name: 'app',
        data() {
            return {
                showModalF: false,
                currentRow: '',
                currentData: '',
                newData: {},
                items: {},
                columns: [
                    'ID', 'ФИО', 'E-mail', 'Телефон', ''
                ]
            }
        },
        components: {
            'modal': {
                template: '#modal-template'
            }
        },
        filters: {
            capitalize: function (str) {
                return str.charAt(0).toUpperCase() + str.slice(1)
            }
        },
        computed: {
            validation: function () {
                let validData = (!this.showModalF) ? this.newData : this.currentData;
                return {
                    fio: !!validData.fio ? validData.fio.trim() : false,
                    email: emailRE.test(validData.email),
                    phone: !!validData.phone ? validData.phone.trim() : false
                }
            },
            isValid: function () {
                let validation = this.validation;
                return Object.keys(validation).every(function (key) {
                    return validation[key]
                })
            }
        },
        methods: {
            getData: function (data) {
                if (!localStorage.getItem('phonebook')) {
                    localStorage.setItem('phonebook', JSON.stringify(data));
                }
                this.items = JSON.parse(localStorage.getItem('phonebook'));
            },
            showModal: function (index) {
                this.currentData = this.items[index];
                this.currentRow = index;
                this.showModalF = true;
            },
            removeRow: function () {
                this.items.splice(this.currentRow, 1);
                localStorage.setItem('phonebook', JSON.stringify(this.items));
                this.closeModal();
            },
            updateRow: function () {
                if (this.isValid) {
                    this.items[this.currentRow] = this.currentData;
                    localStorage.setItem('phonebook', JSON.stringify(this.items));
                    this.closeModal();
                }
            },
            addNewRow: function () {
                if (this.isValid) {
                    this.newData.id = this.items[this.items.length - 1].id + 1;
                    this.items.push(this.newData);
                    localStorage.setItem('phonebook', JSON.stringify(this.items));
                    this.newData = {}
                }
            },
            closeModal: function () {
                this.showModalF = false;
            }
        },
        created: function () {
            this.getData(appData);
        }
    }
</script>

<style>
    body {
        font-family: Helvetica Neue, Arial, sans-serif;
        font-size: 14px;
        color: #444;
    }

    #app {
        width: 850px;
        margin: 0 auto;
    }

    table {
        border: 2px solid #42b983;
        border-radius: 3px;
        background-color: #fff;
    }

    th {
        background-color: #42b983;
        color: rgba(255, 255, 255, 0.66);
        -webkit-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        user-select: none;
    }

    td {
        background-color: #f9f9f9;
    }

    th, td {
        min-width: 120px;
        padding: 10px 20px;
    }

    .modal-mask {
        position: fixed;
        z-index: 9998;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, .5);
        display: table;
        transition: opacity .3s ease;
    }

    .modal-wrapper {
        display: table-cell;
        vertical-align: middle;
    }

    .modal-container {
        width: 300px;
        margin: 0px auto;
        padding: 20px 30px;
        background-color: #fff;
        border-radius: 2px;
        box-shadow: 0 2px 8px rgba(0, 0, 0, .33);
        transition: all .3s ease;
        font-family: Helvetica, Arial, sans-serif;
    }

    .modal-header h3 {
        margin-top: 0;
        color: #42b983;
    }

    .modal-body {
        margin: 20px 0;
    }

    .modal-default-button {
        width: 32%;
    }

    .modal-enter {
        opacity: 0;
    }

    .modal-leave-active {
        opacity: 0;
    }

    .modal-enter .modal-container,
    .modal-leave-active .modal-container {
        -webkit-transform: scale(1.1);
        transform: scale(1.1);
    }

    .default-form p {
        display: flex;
        flex-direction: row;
        justify-content: space-between;
    }

    .default-form label {
        width: 30%;
    }

    .default-form input,
    .default-form span {
        width: 60%;
    }

    .flat-form-wrapper {
        width: 40%;
        margin: 0 auto;
    }

    .flat-form p {
        flex-direction: column;
        justify-content: space-around;
        align-items: center;
    }

    .flat-form label {
        text-align: center;
    }

    .flat-form-wrapper .errors {
        /*display: none;*/
    }
</style>
