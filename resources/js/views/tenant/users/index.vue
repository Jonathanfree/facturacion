<template>
    <div class="users">
        <div class="page-header pr-0">
            <h2><a href="/dashboard"><i class="fas fa-tachometer-alt"></i></a></h2>
            <ol class="breadcrumbs">
                <li class="active"><span>Usuarios</span></li>
            </ol>
            <div class="right-wrapper pull-right">

                <template v-if="showAccessTokenForDiscount">
                    <el-tooltip class="item" content="Genera un token aleatorio para permitir realizar ventas con un porcentaje de descuento superior al límite configurado - Para vendedores" effect="dark" placement="top-start">
                        <button type="button" class="btn btn-info btn-sm  mt-2 mr-2" @click.prevent="clickAccessTokenForDiscount()"><i class="fa fa-check"></i> Generar token</button>
                    </el-tooltip>
                </template>

                <button type="button" class="btn btn-custom btn-sm  mt-2 mr-2" v-if="typeUser == 'admin'" @click.prevent="clickCreate()"><i class="fa fa-plus-circle"></i> Nuevo</button>

                <!--<button type="button" class="btn btn-custom btn-sm  mt-2 mr-2" @click.prevent="clickImport()"><i class="fa fa-upload"></i> Importar</button>-->
            </div>
        </div>
        <div class="card">
            <div class="card-header bg-info">
                <h3 class="my-0">Listado de usuarios</h3>
            </div>
            <div class="card-body">
                <div class="table-responsive">
                    <table class="table">
                        <thead>
                        <tr>
                            <th>#</th>
                            <th>Email</th>
                            <th>Nombre</th>
                            <th>Perfil</th>
                            <th>Api Token</th>
                            <th>Establecimiento</th>
                        </tr>
                        </thead>
                        <tbody>
                        <tr v-for="(row, index) in records" :key="index" :class="!row.active ? 'text-danger' : ''">
                            <td>{{ index + 1 }}</td>
                            <td>
                                {{ row.email }}
                                <sup v-if="row.is_multi_user" style="padding: 0px 3px;border-radius: 4px;" class="bg-info text-white">Multi Usuario</sup>
                            </td>
                            <td>{{ row.name }}</td>
                            <td>{{ row.type }}</td>
                            <td>{{ row.api_token }}</td>
                            <td>{{ row.establishment_description }}</td>
                            <td class="text-right">
                                <button
                                    v-if="typeUser === 'admin' && row.active"
                                    type="button"
                                    class="btn waves-effect waves-light btn-xs btn-info"
                                    @click.prevent="clickCreate(row.id)">
                                    Editar
                                </button>
                                <template v-if="row.id != 1 && typeUser === 'admin'">
                                    <button
                                        v-show="!row.is_multi_user"
                                        type="button"
                                        class="btn waves-effect waves-light btn-xs btn-danger"
                                        @click.prevent="clickDelete(row.id)">
                                        Eliminar
                                    </button>
                                </template>
                                <template v-if="!isMainUser(row.id) && isAdminUser">
                                    <button
                                        v-show="!row.is_multi_user"
                                        type="button"
                                        class="btn waves-effect waves-light btn-xs"
                                        :class="row.active ? 'btn-warning' : 'btn-success'"
                                        @click.prevent="clickActive(row.active, row.id)">
                                        {{ row.active ? 'Inhabilitar' : 'Habilitar'}}
                                    </button>
                                </template>
                            </td>
                        </tr>
                        </tbody>
                    </table>
                </div>
            </div>
            <users-form :showDialog.sync="showDialog"
                        :typeUser="typeUser"
                        :recordId="recordId"></users-form>

            <authorized-token-discount-form :showDialog.sync="showDialogAuthorizedTokenForDiscount" ></authorized-token-discount-form>
        </div>
    </div>
</template>

<script>

    import UsersForm from './form1.vue'
    import AuthorizedTokenDiscountForm from './partials/authorized_token_discount.vue'
    import {deletable} from '../../../mixins/deletable'

    export default {
        props: ['typeUser', 'configuration'],
        mixins: [deletable],
        components: {UsersForm, AuthorizedTokenDiscountForm},
        data() {
            return {
                showDialog: false,
                showDialogAuthorizedTokenForDiscount: false,
                resource: 'users',
                recordId: null,
                records: [],
            }
        },
        created() {
            this.$eventHub.$on('reloadData', () => {
                this.getData()
            })
            this.getData()
        },
        computed:
        {
            showAccessTokenForDiscount()
            {
                return this.typeUser === 'admin' && this.configuration.restrict_seller_discount
            },
            isAdminUser()
            {
                return this.typeUser === 'admin'
            }
        },
        methods: {
            isMainUser(user_id)
            {
                return user_id === 1
            },
            clickAccessTokenForDiscount()
            {
                this.showDialogAuthorizedTokenForDiscount = true
            },
            getData() {
                this.$http.get(`/${this.resource}/records`)
                    .then(response => {
                        this.records = response.data.data
                    })
            },
            clickCreate(recordId = null) {
                this.recordId = recordId
                this.showDialog = true
            },
            clickDelete(id) {
                this.destroy(`/${this.resource}/${id}`).then(() =>
                    this.$eventHub.$emit('reloadData')
                )
            },
            clickActive(active, id)
            {
                this.changeActive(`/${this.resource}/change-active`, { active, id })
                    .then(() => this.$eventHub.$emit('reloadData'))
            },
        }
    }
</script>
