<template>
    <div>
        <h1>Query Builder</h1>
        <nav>
            <router-link :to="{name: 'jobs'}">Check Job Status</router-link>
        </nav>
        <form @submit.stop.prevent="sendQuery()">

            <h2 class="mt-5">Filters</h2>
            <div class="card container py-3 mb-3">
                <template v-for="(clause, index) in queries">
                    <div :key="`clause_${index}`">

                        <div class=" d-flex justify-content-between align-items-end">
                            <div class="form-group">

                                <label>Field</label>
                                <select class="form-control"
                                        v-model="queries[index].field">
                                    <option disabled
                                            selected
                                            :value="''">Choose a search field</option>
                                    <option v-for="field in field_options"
                                            :key="`${field.value}_${index}`"
                                            :value="field.value"
                                            v-text="field.label"></option>
                                </select>
                            </div>
                            <div class="form-group col">

                                <label>Value</label>
                                <input class="form-control"
                                       type="text"
                                       v-model="queries[index].value" />
                            </div>
                            <div class="form-group">
                                <button class="btn btn-danger"
                                        type="button"
                                        @click.stop.prevent="removeQueryClause(index)">X</button>
                            </div>
                        </div>

                        <div v-if="index != queries.length - 1">
                            <div class="form-group">

                                <!-- <label>Operator</label> -->
                                <select class="form-control"
                                        style="width: auto"
                                        v-model="queries[index].operator">
                                    <option disabled
                                            selected
                                            :value="''">Choose an operand</option>
                                    <option v-for="operator in operator_types"
                                            :key="`${operator}_${index}`"
                                            :value="operator"
                                            v-text="operator"></option>
                                </select>
                            </div>
                        </div>
                    </div>
                </template>
                <div>
                    <button class="btn btn-outline-primary"
                            type="button"
                            @click.stop.prevent="addQueryClause()">+ Add Additional Filter</button>
                </div>
            </div>

            <h2 class="mt-5">Output Fields</h2>
            <div class="d-flex">
                <label class="btn btn-sm mr-3 d-flex align-items-center"
                       :class="{
                            'btn-outline-primary': fields_view != 'selected',
                            'btn-primary': fields_view == 'selected'
                        }"><input type="radio"
                           v-model="fields_view"
                           value="selected" />Show Selected Only</label>
                <label class="btn btn-sm d-flex align-items-center"
                       :class="{
                            'btn-outline-primary': fields_view != 'all',
                            'btn-primary': fields_view == 'all'
                        }"><input type="radio"
                           v-model="fields_view"
                           value="all" />Show All Available Fields</label>
                <label @click.stop.prevent="deselectAll"
                       class="btn btn-primary btn-sm d-flex align-items-center ml-auto">Deselect All</label>
            </div>
            <div class="card container">
                <div class="row ">
                    <template v-for="field in fields">
                        <div class="col-3 d-flex align-items-center p-1"
                             :key="`${field}_field`"
                             v-if="fields_view=='all' || selected_fields.indexOf(field) >= 0">
                            <label class="btn"
                                   :class="{
                            'btn-outline-primary': selected_fields.indexOf(field) == -1,
                            'btn-primary': selected_fields.indexOf(field) >= 0,
                        }">
                                <input type="checkbox"
                                       class="mr-2"
                                       :id="`${field}_field`"
                                       v-model="selected_fields"
                                       :value="field" />

                                <span :for="`${field}_field`"
                                      v-text="field"></span>
                            </label>
                        </div>
                    </template>
                    <template v-if="selected_fields.length == 0">
                        <div class="m-3">No output fields are selected.</div>
                    </template>
                </div>
            </div>


            <div class="form-group mt-5">
                <button v-if="selected_fields.length == 0"
                        class="btn btn-primary btn-lg disabled"
                        disabled>Preview Query</button>
                <button v-else
                        class="btn btn-primary btn-lg"
                        type="submit">Get Preview</button>
            </div>

            <div>
                <h2 class="mt-5">Preview Results</h2>
                <table v-if="preview_data"
                       class="table">
                    <tr>
                        <th v-for="field_name in selected_fields"
                            v-text="field_name"
                            :key="`preview_header_${field_name}`"></th>
                    </tr>
                    <template v-for="(row, index) in preview_data">
                        <tr :key="`preview_row_${index}`">
                            <td v-for="field_name in selected_fields"
                                v-text="row[field_name].split('|').join(', ')"
                                :key="`preview_row_${index}_${field_name}`"></td>
                        </tr>
                    </template>
                    <tr v-if="preview_data.length == 0">
                        <td :colspan="selected_fields.length">
                            No results were found. Please modify your query and try again.
                        </td>
                    </tr>
                </table>
                <div v-else>
                    There are currently no preview results.
                </div>
                <!-- <pre class="pre"
                     v-text="result"></pre> -->
            </div>

            <div class="form-group mt-5">
                <!-- <button v-if="selected_fields.length == 0 || !preview_data"
                        class="btn btn-primary btn-lg disabled"
                        disabled
                        @click.stop.prevent>Submit Query</button>
                <button v-else
                        @click.stop.prevent="sendQuery(true)"
                        class="btn btn-primary btn-lg"
                        type="submit">Submit Query</button> -->
                <button @click.stop.prevent="sendQuery(true)"
                        class="btn btn-primary btn-lg"
                        type="submit">Submit Query</button>
            </div>
        </form>

        <template v-if="error_message">
            <div class="modal show"
                 style="display: block;"
                 tabindex="-1"
                 role="dialog">
                <div class="modal-dialog "
                     role="document">
                    <div class="modal-content">
                        <div class="alert alert-danger mb-0">
                            <div class="modal-header">
                                <h5 class="modal-title">There was a problem with your query</h5>
                                <button type="button"
                                        class="close"
                                        @click="error_message = ''"
                                        aria-label="Close">
                                    <span aria-hidden="true">&times;</span>
                                </button>
                            </div>
                            <div class="modal-body">
                                <p v-text="error_message"></p>
                            </div>
                            <div class="modal-footer">
                                <button type="button"
                                        class="btn btn-secondary"
                                        @click="error_message = ''">OK</button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="modal-backdrop fade show"
                 @click="error_message = ''"></div>
        </template>

        <template v-if="query_modal_open && !is_loading">
            <div class="modal show"
                 style="display: block;"
                 tabindex="-1"
                 role="dialog">
                <div class="modal-dialog "
                     role="document">
                    <div class="modal-content">
                        <div class="alert alert-success mb-0">
                            <div class="modal-header">
                                <h5 class="modal-title">Query Is Running</h5>
                                <button type="button"
                                        class="close"
                                        @click="query_modal_open = false;"
                                        aria-label="Close">
                                    <span aria-hidden="true">&times;</span>
                                </button>
                            </div>
                            <div class="modal-body">
                                <div>
                                    Job ID: <b v-text="query_results.job_id"></b>
                                </div>
                                <button @click.prevent.stop="$router.push({name: 'jobs'})"
                                        class="btn btn-primary">Check Job Statuses</button>
                            </div>
                            <div class="modal-footer">
                                <button type="button"
                                        class="btn btn-secondary"
                                        @click="query_modal_open = false;">OK</button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="modal-backdrop fade show"
                 @click="query_modal_open = false;"></div>
        </template>

    </div>
</template>
<script>
let field_options = [
    // { value: "wosId", label: "WoS ID" },
    { value: "year", label: "Year" },
    { value: "authorsFullName", label: "Author" },
    { value: "journalsName", label: "Journal Name" },
    { value: "title", label: "Title" }
];
let operator_types = [
    "AND",
    "OR"
    // "NOT"
];

export default {
    data: function() {
        return {
            result: null,
            database_status: {},
            status: {},
            selected_fields: [
                "wosId",
                "year",
                "authorsFullName",
                "journalName"
            ],
            queries: [
                {
                    field: "",
                    value: "",
                    join: ""
                }
            ],
            preview_result: null,
            fields_view: "selected",
            error_message: "",
            query_results: {
                job_id: "",
                message_id: "",
                s3_location: ""
            },
            query_modal_open: false
        };
    },
    computed: {
        is_loading: function(){
            return this.isLoading;
        },
        fields: function() {
            return this.$store.getters["query/validFields"];
        },
        field_options: function() {
            return field_options;
        },
        operator_types: function() {
            return operator_types;
        },
        preview_data: function() {
            let tmp =
                (this.result && this.result.data && this.result.data.wos) ||
                null;
            if (tmp) {
                tmp.splice(9);
            }
            return tmp;
        }
    },
    methods: {
        deselectAll: function() {
            this.selected_fields.splice(0, this.selected_fields.length);
        },
        addQueryClause: function(clause) {
            let len = this.queries.length;

            //when adding a new clause, set the operator of the previous clause
            if (len) {
                this.queries[len - 1].operator = operator_types[0];
            }

            this.queries.push({
                field: "",
                value: "",
                operator: ""
            });
        },
        removeQueryClause: function(index) {
            this.queries.splice(index, 1);
        },
        getStatus: function() {
            // //gets the status of the data api
            // let status_prom = this.$cadre.axios({
            //     url: "/data/status"
            // });
            // status_prom.then(
            //     result => {
            //         this.status = result.data;
            //     },
            //     err => {
            //         this.status = "Could not get status\n" + err;
            //     }
            // );

            // //gets the status of the wos database
            // let database_prom = this.$cadre.axios({
            //     url: "/data/wos/status"
            // });
            // database_prom.then(
            //     result => {
            //         this.database_status = result.data;
            //     },
            //     err => {
            //         this.database_status = "Could not get status\n" + err;
            //     }
            // );
        },
        sendQuery: function(async) {
            async = async || false;
            let query = [];

            //remove empty clauses from query by creating a new array
            for (let clause of this.queries) {
                if (clause.field && clause.value) {
                    query.push({
                        argument: clause.field,
                        value: clause.value,
                        operator: clause.operator
                    });
                }
            }

            if (query.length === 0) {
                this.error_message = "You must provide at least one filter.";
                console.error("Empty Query", this.queries);
                return false;
            }

            //remove the operator from the last clause so that it doesn't confuse the data api
            let last_clause = query[query.length - 1];
            last_clause.operator = "";

            let payload = {
                async: async,
                query: query,
                output_fields: [...this.selected_fields],
                dataset: "wos"
            };

            this.result = null;
            if (!async) {
                this.$emit("startLoading", {
                    message: "Fetching Preview...",
                    key: "query"
                });
            } else {
                this.$emit("startLoading", {
                    message: "Sending Query...",
                    key: "query"
                });
            }

            let query_prom = this.$store.dispatch("query/sendQuery", payload);

            query_prom.then(
                result => {
                    this.$emit("stopLoading", { key: "query" });
                    if (!async) {
                        if(result.errors)
                        {
                            console.error("GraphQL Error: ", result.errors);
                            this.error_message = "";
                            for(let error of result.errors)
                            {
                                this.error_message += error.message + " ";
                            }
                            this.result = null;
                        }
                        else
                        {
                            this.result = result;
                        }
                    } else {
                        if (result[0] && result[1] === 200) {
                            this.query_results.job_id = result[0].job_id;
                            this.query_results.message_id =
                                result[0].message_id;
                            this.query_results.s3_location =
                                result[0].s3_location;
                            this.query_modal_open = true;
                        }
                    }
                },
                error => {
                    this.$emit("stopLoading", { key: "query" });
                    // console.error(error);
                    if (error.code === 1000) {
                        if (async) {
                            this.error_message =
                                "Your query timed out.  Please try again in a moment.";
                        } else {
                            this.error_message =
                                "Your preview query timed out.  Please try again in a moment.";
                        }
                    }
                    else if (error.response)
                    {
                        // console.debug(error.response);
                        if(error.response.status == 401)
                        {
                            this.error_message = "You do not have access to this dataset.";
                        }
                        else
                        {
                            this.error_message = error.response.data.error.toString();
                        }
                    }
                    else {
                        this.error_message = error.toString();
                    }
                }
            );
        }
    },
    props: {
        isLoading: Number
    },
    mounted: function() {
        this.getStatus();
    }
};
</script>
<style lang="scss">
</style>

