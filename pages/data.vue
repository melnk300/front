<template>
    <div class="container">
        <div class="table">
            <div class="table_header">
                <p v-for="table, index in tables" :class="{'active_table': activeTable === index}" v-on:click="activeTable = index">{{table}}</p>
                <p class="new_view" v-on:click="showNewViews">+</p>
            </div>
            <div class="table_inner">
                <table>
                    <thead>
                        <td></td>
                        <td v-for="column in data[activeTable]?.columns">{{ column.field }}</td>
                    </thead>
                    <tbody>
                        <tr v-for="row, index in data[activeTable]?.data">
                            <td>{{ index + 1 }}</td>
                            <td v-for="column in row.slice(1)">{{ column }}</td>
                        </tr>
                        <tr>
                            <td colspan="100%">Новая запись +</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
    <Transition name="new_view">
        <div class="new_view_pop" v-if="isNewViewShowed" v-on-click-outside="closeNewView">
            <form class="new">
                <input type="text" placeholder="Название" v-model="datasetName">
                <select name="dataset" placeholder="Набор данных" v-model="dataset">
                    <option value="def" disabled>Набор данных</option>
                    <option value="client">Клиенты</option>
                    <option value="teacher">Педагоги</option>
                    <option value="lesson">Занятия</option>
                    <option value="group">Группы</option>
                </select>
                <div class="datafields">
                    <div class="fieldsettings" v-for="field in fields[dataset]">
                        <label>
                            <input type="checkbox" name="datafield" :value="field" v-model="field.state">
                            {{ field.field }}
                        </label>
                        <div v-if="field.state">
                            <select name="datafield_setting" v-model="field.option">
                                <option value="empty" default>Без условий</option>
                                <template v-if="field.type === 'string'">
                                    <option value="contains">~~</option>
                                    <option value="not_contains">!~~</option>
                                    <option value="equal">=</option>
                                    <option value="not_equal">!=</option>
                                </template>
                                <template v-if="field.type === 'number'">
                                    <option value="equal">=</option>
                                    <option value="not_equal">!=</option>
                                    <option value="more">></option>
                                    <option value="less">&lt;</option>
                                    <option value="more_equal">>=</option>
                                    <option value="less_equal">&lt;=</option>
                                </template>
                            </select>
                            <input type="text" placeholder="Значение" v-model="field.value" v-if="!(['empty', ''].includes(field.option || ''))">
                        </div>
                    </div>
                </div>
                <div class="form_btns">
                    <button class="save" v-on:click="saveDataSet">Сохранить</button>
                    <button class="cancel" v-on:click="closeNewView">Отмена</button>
                </div>
            </form>
        </div>
    </Transition>
</template>

<script setup lang="ts">
    import { vOnClickOutside } from '@vueuse/components'

    interface Table {
        columns: Column[]
        data: any[]
    }
    
    interface Column {
        field: string,
        type: string
    }

    interface StateColumn extends Column {
        state: boolean
    }

    interface FieldSetting {
        option?: string,
        value?: string | number
    }

    let activeTable = ref(0)

    let dataset = ref()

    const datasetName = ref("")

    const tables = ref<string[]>(["Клиенты", "Оплата за последний месяц", "Пробники"])
    const data = ref<Table[]>([
        {
            columns: [
                {field: "Имя", type: "string"},
                {field: "Телефон", type: "string"},
                {field: "Состояние", type: "string"},
            ],
            data: [
                [4, "Руслан Мельник", "+79821352159", "Пробный прошёл"],
                [2, "Дмитрий Матвеев", "+79821352159", "Мег Сб 18:00"],
                [3, "Василий Гусельников", "+79821352159", "Интер Вс 14:00"]
            ]
        }, {
            columns: [
                {field: "Имя", type: "string"},
                {field: "Группа", type: "string"},
                {field: "Дата", type: "string"},
                {field: "Сумма", type: "number"},
            ], 
            data: [
                [1, "Руслан Мельник", "Мег Сб 18:00", "22.06.2023", 5400],
                [2, "Дмитрий Матвеев", "Мег Сб 18:00", "22.06.2023", 3600],
                [3, "Василий Гусельников", "Интер Вс 14:00", "22.06.2023", 5800]
            ]
        }, {
            columns: [
                {field: "Имя", type: "string"},
                {field: "Телефон", type: "string"},
                {field: "Состояние", type: "string"},
                {field: "Офис", type: "string"},
                {field: "Дата", type: "string"},
            ],
            data: [
                [1, "Руслан Мельник", "+79821352159", "Пробный прошёл", "Мег", "24.06.2023"],
                [2, "Шамиль Хайрутдинов", "+79821352159", "Пробный записан", "Мег",  "24.06.2023"],
                [3, "Илья Медведев", "+79821352159", "Пробный Перенесён", "Интер", "22.06.2023"]
            ]
        }
    ])

    let fields = ref<{[data: string]: (StateColumn & FieldSetting)[]}>({
        "client": [
            {
                field: "Имя",
                type: "string",
                state: false,
            }, {
                field: "Телефон",
                type: "string",
                state: false
            }, {
                field: "Имя родителя",
                type: "string",
                state: false
            }, {
                field: "Телефон родителя",
                type: "string",
                state: false
            }, {
                field: "Состояние",
                type: "string",
                state: false
            }, {
                field: "Группа",
                type: "string",
                state: false
            }, {
                field: "Педагог",
                type: "string",
                state: false
            }, {
                field: "Баланс",
                type: "number",
                state: false
            }, {
                field: "Пропущенно занятий",
                type: "number",
                state: false
            }
        ], 
        "teacher": [
            {
                field: "Имя",
                type: "string",
                state: false
            }, {
                field: "Телефон",
                type: "string",
                state: false
            }, {
                field: "Ставка",
                type: "number",
                state: false
            }, {
                field: "Email",
                type: "string",
                state: false
            }, {
                field: "Группы",
                type: "string",
                state: false
            },
        ], "lesson": [
            {
                field: "Группа",
                type: "string",
                state: false
            },
            {
                field: "Дата",
                type: "string",
                state: false
            },
            {
                field: "Время",
                type: "string",
                state: false
            },
            {
                field: "Педагог",
                type: "string",
                state: false,
            },
            {
                field: "Кабинет",
                type: "string",
                state: false
            },
            {
                field: "Состояние",
                type: "string",
                state: false
            },
        ]
    })

    let isNewViewShowed = ref(false)

    const closeNewView = () => { isNewViewShowed.value = false }

    const showNewViews = () => { isNewViewShowed.value = true }

    const saveDataSet = (e: Event) => {
        e.preventDefault()
        console.log(dataset.value)
        console.log(fields.value[dataset.value])
        closeNewView()
        // TO-DO
        // axios.post(`${process.env.docker_internal}:8080/data/`, {
        //     dataset: dataset.value,
        //     fields: fields.value[dataset.value]
        // })
        tables.value.push(datasetName.value)
        data.value.push({
            columns: fields.value[dataset.value].filter((field) => field.state),
            data: []
        })



    }

</script>

<style scoped lang="less">
    .container {
        margin: 10px;
        position: relative;
        width: 100vw;
    }

    .table_header {
        display: flex;
        gap: 10px;

        p {
            cursor: pointer;
            position: relative;

            &::after {
                content: " ";
                position: absolute;
                bottom: -2px;
                left: 0;
                width: 0;
                height: 2px;
                background-color: #333;
                opacity: 0;
                transition: 0.3s;
            }

            &.new_view {
                padding: 0 10px;
            }
        }

        .active_table::after {
            content: " ";
            position: absolute;
            bottom: -2px;
            left: 0;
            width: 100%;
            height: 2px;
            background-color: #333;
            opacity: 1;
        }
    }    

    table {
        border-spacing: 0;
    
        td {
            border: 1px solid #e0e0e0;
            padding: 4px;
        }
    }

    .new_view_pop {
        position: absolute;
        top: 0;
        right: 0;
        width: 30vw;
        background-color: #fff;
        border: 1px solid #e0e0e0;
        border-top: none;
        padding: 10px;
        z-index: 1;
    }
    
    .new_view-enter-active, .new_view-leave-active {
        transition: all 0.5s;
    }

    .new_view-enter-from, .new_view-leave-to {
        transform: translateX(30vw);
    }

    .new_view_pop {
        padding: 10px;
        border-top-left-radius: 10px;
        border-bottom-left-radius: 10px;

        .new {
            display: flex;
            flex-direction: column;
            gap: 10px;

            input, select {
                padding: 10px;
                border: 1px solid #e0e0e0;
                border-radius: 5px;
            }
        }
    }

    .datafields {
        display: flex;
        flex-direction: column;
        gap: 10px;
    }

    .fieldsettings {
        display: flex;
        gap: 5px;
        align-items: center;
    }

    .form_btns {
        display: flex;
        justify-content: flex-end;
        gap: 10px;
        margin-top: 10px;

        button {
            padding: 10px;
            border: 1px solid #e0e0e0;
            border-radius: 5px;
            cursor: pointer;
        }

        .save {
            background-color: #4caf50;
            color: #fff;
        }

        .cancel {
            background-color: #f44336;
            color: #fff;
        }
    }
</style>