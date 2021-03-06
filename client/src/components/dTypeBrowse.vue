<template>
    <div>
        <v-toolbar flat color='white'>
            <v-spacer></v-spacer>
            <v-dialog v-model='dialog' max-width='600px'>
                <template v-slot:activator='{ on }'>
                    <v-btn
                        small flat fab
                        v-on='on'
                    >
                        <v-icon>fa-plus</v-icon>
                    </v-btn>
                </template>
                <v-card>
                    <v-card-title>
                        <span class='headline'>{{ formTitle }}</span>
                    </v-card-title>

                    <v-card-text v-if="editedItem">
                        <v-container grid-list-md>
                            <v-layout wrap>
                                <template v-for="header in headers">
                                    <v-flex xs12
                                        v-if="header.value === 'types' || header.value === 'optionals' || header.value === 'outputs'"
                                    >
                                        <v-layout row wrap>
                                            <dTypeTypeEdit
                                                v-for="(type, i) in editedItem[header.value]"
                                                :type="type"
                                                v-on:change="onChangedType(header.value, $event, i)"
                                                v-on:remove="onRemovedType(header.value,type, i)"
                                            />
                                            <v-flex xs12>
                                                <dTypeSearch
                                                    :label='header.value'
                                                    itemKey='name'
                                                    itemValue='name'
                                                    :items='items'
                                                    v-on:change="onSelectedType(header.value, $event)"
                                                />
                                            </v-flex>
                                        </v-layout>
                                    </v-flex>
                                    <v-flex xs12 v-else>
                                        <v-text-field v-model='editedItem[header.value]' :label='header.value'></v-text-field>
                                    </v-flex>
                                </template>
                            </v-layout>
                        </v-container>
                    </v-card-text>

                    <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn color='blue darken-1' flat @click='close'>Cancel</v-btn>
                        <v-btn color='blue darken-1' flat @click='save'>Save</v-btn>
                    </v-card-actions>
                </v-card>
            </v-dialog>
            <v-dialog v-model='dialogBulk' max-width='600px'>
                <template v-slot:activator='{ on }'>
                    <v-btn
                        small flat fab
                        v-on='on'
                    >
                        <v-icon>fa-folder-plus</v-icon>
                    </v-btn>
                </template>
                <v-card>
                    <v-card-title>
                        <span class='headline'>Bulk Insert</span>
                    </v-card-title>

                    <v-card-text>
                        <v-container grid-list-md>
                            <v-layout wrap>
                                <v-flex xs12>
                                    <v-textarea
                                        outline
                                        name="bulkInsert"
                                        v-model='bulkInsert'
                                        label="array of types"
                                        :placeholder="bulkInsertDefault"
                                    ></v-textarea>
                                </v-flex>
                            </v-layout>
                        </v-container>
                    </v-card-text>

                    <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn color='blue darken-1' flat @click='closeBulk'>Cancel</v-btn>
                        <v-btn color='blue darken-1' flat @click='saveBulk'>Save</v-btn>
                    </v-card-actions>
                </v-card>
            </v-dialog>
        </v-toolbar>
        <v-data-table
            :headers='headers'
            :items='items'
            class='elevation-1'
            :pagination.sync="paginationBrowse"
        >
            <template v-slot:items='props'>

                <template
                    v-for="header in headers"
                >
                    <td class='text-xs-left'>
                        <dTypeBrowseField
                            :type="header.type"
                            :value="props.item[header.value]"
                        />
                    </td>
                </template>
                <td class='justify-center layout px-0'>
                    <v-btn
                        flat icon small
                        color="blue darken-4"
                        :to="`/dtype/${props.item.lang}/${props.item.name}`"
                    >
                        <v-icon>link</v-icon>
                    </v-btn>

                    <v-btn
                        flat icon small
                        color="grey darken-2"
                        @click='editItem(props.item)'
                    >
                        <v-icon>edit</v-icon>
                    </v-btn>
                    <v-btn
                        flat icon small
                        color="grey darken-2"
                        @click='deleteItem(props.item)'
                    >
                        <v-icon>delete</v-icon>
                    </v-btn>
                </td>
            </template>
        </v-data-table>
    </div>
</template>

<script>
import dTypeSearch from '../components/dTypeSearch';
import dTypeBrowseField from '../components/dTypeBrowseField';
import dTypeTypeEdit from '../components/dTypeTypeEdit';

export default {
    name: 'dTypeBrowse',
    props: ['headers', 'items', 'defaultItem'],
    components: {
        dTypeSearch,
        dTypeBrowseField,
        dTypeTypeEdit,
    },
    data() {
        let data = {
            dialog: false,
            dialogBulk: false,
            editedIndex: -1,
            editedItem: {},
            bulkInsertDefault: '{}',
            bulkInsert: '{}',
            paginationBrowse: {rowsPerPage: 25},

        }
        return data;
    },
    computed: {
        formTitle() {
            return this.editedIndex === -1 ? 'New Item' : 'Edit Item';
        },
    },
    watch: {
        dialog (val) {
            val || this.close();
        },
        dialogBulk (val) {
            val || this.closeBulk();
        },
        items() {
            this.setBulkInsert();
        },
        defaultItem() {
            this.setDefaults();
        }
    },
    methods: {
        setDefaults() {
            this.editedItem = Object.assign({}, this.defaultItem);
            this.bulkInsertDefault = JSON.stringify([this.defaultItem]);
            this.bulkInsert = this.items.length > 0 ? JSON.stringify(this.items) : this.bulkInsertDefault;
        },
        setBulkInsert() {
            this.bulkInsert = JSON.stringify(
                this.items.map(dt => {
                    let obj = {};
                    Object.keys(this.defaultItem).forEach(key => {
                        obj[key] = dt[key];
                    });
                    obj.typeHash = dt.typeHash;
                    return obj;
                })
            );
        },
        async insert(dtype) {
            this.$emit('insert', dtype);
        },
        async batchInsert(dtypeArray) {
            this.$emit('batchInsert', dtypeArray);
        },
        async update(dtype) {
            this.$emit('update', dtype);
        },
        async delete(dtype) {
            this.$emit('remove', dtype);
        },
        editItem(obj) {
            let item = Object.assign({}, obj);
            Object.keys(item)
                .filter(key => !Number(key) && Number(key) != 0)
                .forEach((key) => {
                    const header = this.headers.find(header => header.value === key);
                    if (header) {
                        let dtype = this.$store.state.dtypes.find(dtype => dtype.name === header.type.name);

                        if (dtype && dtype.types.length && ['types', 'optionals', 'outputs'].indexOf(key) === -1 ) {
                            item[key] = JSON.stringify(item[key]);
                        }

                    }
                });
            if (item.types) {
                item.types = item.types.map(type => {
                    type.dimensions = JSON.stringify(type.dimensions);
                    return type;
                });
                item.optionals = item.optionals.map(type => {
                    type.dimensions = JSON.stringify(type.dimensions);
                    return type;
                });
                item.outputs = item.outputs.map(type => {
                    type.dimensions = JSON.stringify(type.dimensions);
                    return type;
                });
            }
            this.editedIndex = this.items.indexOf(obj);
            this.editedItem = Object.assign({}, item);
            this.dialog = true;
        },
        deleteItem(item) {
            const index = this.items.indexOf(item)
            confirm('Are you sure you want to delete this item?') && this.delete(item);
        },
        close() {
            this.dialog = false;
            setTimeout(() => {
                this.editedItem = Object.assign({}, this.defaultItem);
                this.editedIndex = -1;
            }, 300)
        },
        save() {
            this.headers.forEach((header) => {
                // If the type is an array, and value is a string, we need to split
                if (header.type.dimensions.length > 0) {
                    if (typeof this.editedItem[header.value] === 'string') {
                        this.editedItem[header.value] = JSON.parse(this.editedItem[header.value]);
                    }
                }// else {
                    let dtype = this.$store.state.dtypes.find(dtype => dtype.name === header.type.name);

                    if (dtype && dtype.types.length && typeof this.editedItem[header.value] == 'string') {
                        this.editedItem[header.value] = JSON.parse(this.editedItem[header.value]);
                    }
                //}
            });
            if (this.editedItem.types) {
                this.editedItem.types = this.editedItem.types.map(type => {
                    type.dimensions = JSON.parse(type.dimensions);
                    return type;
                });
                this.editedItem.optionals = this.editedItem.optionals.map(type => {
                    type.dimensions = JSON.parse(type.dimensions);
                    return type;
                });
                this.editedItem.outputs = this.editedItem.outputs.map(type => {
                    type.dimensions = JSON.parse(type.dimensions);
                    return type;
                });
            }
            if (this.editedIndex > -1) {
                this.update(this.editedItem);
            } else {
                this.insert(this.editedItem);
            }
            this.close();
        },
        closeBulk() {
            this.dialogBulk = false;
        },
        saveBulk() {
            let bulk;
            try {
                bulk = JSON.parse(this.bulkInsert);
                this.batchInsert(bulk);
                this.closeBulk();
            } catch (e) {
                alert(`${e}`);
            }
        },
        onSelectedType(key, values) {
            values = this.editedItem[key].concat(values.map((value) => {
                return {name: value, label: '', relation: 0, dimensions: []};
            }));
            this.editedItem[key] = values;
        },
        onChangedType(key, changed, i) {
            this.editedItem[key][i][changed[0]] = changed[1];
        },
        onRemovedType(key, value, i) {
            this.editedItem[key].splice(i, 1);
        },
    },
};
</script>

<style>
.container {
    max-width: 100%;
}
</style>
