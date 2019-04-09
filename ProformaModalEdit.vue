<template>
	<div>
		<!-- Modal -->
		<div :id="modal_id" class="modal fade" role="dialog" tabindex="-1" ref="editproformamodal">
			<div class="modal-dialog bamboo-modal-lg">

				<!-- Modal content-->
				<div class="modal-content">
					<div class="modal-header">
						<h4 class="modal-title">
							<span>Proforma: {{ form.proforma_name }}</span>
							<span v-show="form.changes.any()" class="label label-warning pull-right m-l-10">Changes are made</span>
							<span v-if="!isLocked && !cancelled" class="label label-success pull-right">{{ edit_message }}</span>
							<span v-if="isLocked && !cancelled" class="label label-danger pull-right">{{ edit_message }}</span>
						</h4>
					</div>
					<form>
						<div class="modal-body">
							<slot name="body">
								<div class="row">
									<div class="col-md-6">
										<div class="col-md-3">
											<label for="proforma-number" class="bamboo-modal-label">Proforma:</label>
											<input disabled class="form-control" type="text" v-model="form.proforma_name">
										</div>
										<div class="col-md-3">
											<label for="purchase-order-number" class="bamboo-modal-label">Status:</label>
											<v-select
												:options="proforma_status_options"
												v-model="form.proforma_status"
												:disabled="disableStatusField || cancelled"
												@select2selecting="statusBeforeChange"
												selectID="proforma-status"
											>
											</v-select>
										</div>
										<div class="col-md-6">
											<label for="purchase-order-number" class="bamboo-modal-label">Intern Comment:</label>
											<input :disabled="closed || cancelled || isLocked" class="form-control" type="text" v-model="form.intern_comment">
										</div>

										<div class="col-md-3">
											<label for="purchase-order-number" class="bamboo-modal-label">Date Created:</label>
											<input disabled class="form-control" type="text" v-model="form.date_created">
										</div>
										<div class="col-md-3">
											<label for="purchase-order-number" :class="form.errors.has('proforma_dealer') ? 'label label-danger' : 'bamboo-modal-label'">Dealer:</label>
											<v-select
												:disabled="disableDealerField || isLocked || cancelled"
												:options="proforma_dealer_options"
												v-model="form.proforma_dealer"
												:selectOnClose="true"
												placeholder="Choose dealer">
											</v-select>
										</div>
										<div class="col-md-6">
											<label for="purchase-order-number" class="bamboo-modal-label">Comment:</label>
											<input :disabled="closed || cancelled || isLocked" class="form-control" type="text" v-model="form.comment">
										</div>

										<div class="col-md-3">
											<label class="bamboo-modal-label">Date on Document:</label>
											<div class="input-group date" id="edit-proforma-document-date">
											    <input :disabled="closed || cancelled || isLocked" type="text" class="form-control" id="document-date" v-model="form.document_date">
											    <div class="input-group-addon datepicker-button">
											        <span class="glyphicon glyphicon-th"></span>
											    </div>
											</div>
										</div>
										<div class="col-md-3">
											<label :class="form.errors.has('shipping_from') ? 'label label-danger' : 'bamboo-modal-label'">Shipping from:</label>
											<v-select
												:options="proforma_shipping_from_options"
												v-model="form.shipping_from"
												placeholder="Shipping From"
												:disabled="closed || cancelled || isLocked"
											>
											</v-select>
										</div>
										<div class="col-md-6">
											<label class="bamboo-modal-label">User Field:</label>
											<input :disabled="closed || cancelled || isLocked" class="form-control" type="text" v-model="form.user_field">
										</div>
									</div>
									<div class="col-md-6">
										<div class="col-md-8">
											<div class="col-md-6">
												<label
													:class="'bamboo-modal-label' + ( form.shipping_address_overwritten ? ' text-danger' : '' )"
												>
													{{ shipping_address_title }}:
												</label>
												<textarea :disabled="closed || cancelled || isLocked" class="form-control" rows="5" v-model="form.shipping_address"></textarea>
											</div>
											<div class="col-md-6">
												<label
													:class="'bamboo-modal-label' + ( form.invoice_address_overwritten ? ' text-danger' : '' )"
												>
													{{ invoice_address_title }}:
												</label>
												<textarea :disabled="closed || cancelled || isLocked" class="form-control" rows="5" v-model="form.invoice_address"></textarea>
											</div>										
										</div>
										<div class="col-md-4">
											<div class="row">
												<div class="col-md-12">
													<label class="bamboo-modal-label">Document VAT:</label>
													<v-select :disabled="closed || cancelled || isLocked" :options="taxes_options" v-model="form.tax"></v-select>
												</div>
												<div class="col-md-12">
													<label class="bamboo-modal-label">Add Items to Shipping document:</label>
													<div class="input-group">
														<div class="input-group-btn">
															<button type="button"
																@click="addItemsToShippingDocument"
																:disabled="disableAddToShippingBtn || isLocked"
																class="btn btn-default glyphicon glyphicon-plus"
																style="margin-top:-1px;"
																id="add-to-shipping-btn">
															</button>
														</div>
														<v-select
															:disabled="disableAddToShippingBtn || isLocked"
															:options="open_shippings_options"
															v-model="open_shipping"
															placeholder="New Shipping Doc."
															:allowClear="true"
														>
														</v-select>
													</div>
												</div>
											</div>								
										</div>										
									</div>
								</div>

								<hr class="bamboo-hr mar-top-20">
		
									<div class="row">
										<div class="col-md-10">
											<table class="table table-nonfluid table-condensed">
												<thead>
													<tr>
														<th>Item</th>
														<th v-show="!reservableItem">Qty</th>
														<th v-show="!reservableItem" :class="form.errors.has('model') ? 'text-danger' : ''">Model</th>
														<th v-show="!reservableItem" :class="form.errors.has('size') ? 'text-danger' : ''">Size</th>
														<!-- <th v-show="!reservableItem">SN</th> -->
														<th v-show="!reservableItem" :class="form.errors.has('color') ? 'text-danger' : ''">Color</th>
														<th v-show="!reservableItem">Unit Price</th>
														<th v-show="!reservableItem">Total</th>
														<!-- <th v-show="!reservableItem">Shipping</th> -->
														<th v-show="!reservableItem">
															<button class="btn btn-default" type="button" @click="addQuantityItemToTable" :disabled="form.errors.any()">
																<span class="glyphicon glyphicon-plus"></span>
															</button>
														</th>
													</tr>
												</thead>

												<tbody>
													<tr>
														<td width="30px">
															<v-select
																:disabled="closed || cancelled || isLocked"
																:options="product_type_options"
																v-model="product_type"
																placeholder="Product Type">
															</v-select>
														</td>
														<td v-show="!reservableItem">
															<!-- Quantity -->
															<input type="text" class="form-control new-item-quantity" v-model="default_quantity">
														</td>
														<td v-show="!reservableItem">
															<!-- Model -->
															<v-select :options="product_model_options" placeholder="Product Model" v-model="product_model"></v-select>
														</td>
														<td v-show="!reservableItem">
															<!-- Size -->
															<v-select :disabled="item_size_disabled" :options="product_size_options" placeholder="Product Size" v-model="product_size"></v-select>
														</td>

														<!-- <td v-show="!reservableItem"> -->
															<!-- Serial Number -->
															<!-- <input disabled="" type="text" class="form-control"> -->
														<!-- </td> -->

														<td v-show="!reservableItem">
															<!-- Color -->
															<v-select :disabled="item_color_disabled" :options="product_color_options" placeholder="Product Color" v-model="product_color"></v-select>
														</td>
														<td v-show="!reservableItem">
															<!-- Unit Price -->
															<input type="text" class="form-control new-item-price" v-model="default_price">
														</td>
														<td v-show="!reservableItem">
															<!-- Total -->
															<input disabled="" type="text" class="form-control new-item-total" v-model="new_product_total">
														</td>
														<!-- <td v-show="!reservableItem"> -->
															<!-- Shipping -->
															<!-- <v-select placeholder="Product Type"></v-select> -->
														<!-- </td> -->
														<td></td>
													</tr>
												</tbody>
											</table>
										</div>
									</div>

								<hr class="bamboo-hr">

								<div class="row">
									<div class="col-md-12">
										<table class="table table-hover table-condensed new-proforma-table">
											<thead>
												<tr>
													<th>
														<input v-show="closed || cancelled || isLocked"
															:disabled="form.productsTableList.length == 0 || isLocked"
															id="selectAllCheckbox"
															type="checkbox"
															@click="selectAllItems"
															v-model="selectAll">
													</th>
													<th>Item</th>
													<th>Qty</th>
													<th>Model</th>
													<th>Size</th>
													<th>Color</th>
													<th>SN</th>
													<th>Location</th>
													<th>Unit Price</th>
													<!-- <th>Discount</th> -->
													<th>Total</th>
													<th>Shipping</th>
													<th>Invoice</th>
													<th></th>
												</tr>
											</thead>

											<tbody>
												<template v-if="cancelled" v-for="item in form.cancelled_items">
													<tr @click="rowClicked(item, item.class)" :class="item.class">
														<td @click.stop class="cursor-pointer paddingless-side" align="center">
															<input type="hidden" v-model="item.selected">
															<span v-if="item.cancelled_combos && item.cancelled_combos.length"
																:class="'glyphicon glyphicon-chevron-' + (item.combos_show ? 'up' : 'down')"
																@click="toggleCombo(item)"
															>
															</span>
														</td>
														<td>{{ item.type }}</td>
														<td class="table-td">
															<input type="text"
																:disabled="true"
																class="table-input-qty"
																v-model="item.quantity"
															>
														</td>
														<td>{{ item.name }}</td>
														<td>{{ item.size }}</td>
														<td>{{ item.color }}</td>
														<td>{{ item.serial }}</td>
														<td>{{ item.location }}</td>
														<td class="table-td">
															<!-- Unit Price -->
															<input type="text"
																class="table-input"
																:disabled="true"
																v-model="item.unit_price"
															>
														</td>
														<!-- <td>
															Discount
															<input type="text"
																class="table-input"
															>
														</td> -->
														<td>{{ (item.quantity * item.unit_price + (item.quantity * item.unit_price * (item.tax_value/100))).toFixed(2) }}</td>
														<td><a data-toggle="modal" data-target="#edit-shipping-modal" :id="item.related_shipping_id" data-backdrop="static" data-dismiss="modal" href="#">{{ item.related_shipping_document }}</a></td>
														<td><a data-toggle="modal" data-target="#edit-invoice-modal" :id="item.related_invoice_id" data-backdrop="static" data-dismiss="modal" href="#">{{ item.related_invoice_document }}</a></td>
														<td>
															<!-- Remove Button not available on cancelled document -->
														</td>
													</tr>
													<transition name="combo">
														<tr v-if="item.cancelled_combos && item.cancelled_combos.length && item.combos_show" :class="item.class">
															<td colspan="12">
																<div class="row">
																	<div class="col-md-6 col-md-offset-1">
																		<div class="panel panel-primary panel-paddingless combos-table">
																			<div class="panel-heading" style="height: 20px;padding:0">
																				{{item.name}} ({{ item.serial }}) Accessories
																			</div>
																			<div class="panel-body">
																				<table class="table table-striped table-condensed new-proforma-table">
																					<thead>
																						<tr>
																							<!-- <th>
																								<input :disabled="form.productsTableList.length == 0" id="selectAllCheckbox" type="checkbox" @click="selectAllItems" v-model="selectAll">
																							</th> -->
																							<th>Item</th>
																							<th>Qty</th>
																							<th>Model</th>
																							<th>Size</th>
																							<th>Color</th>
																							<th>Unit Price</th>
																							<th>Total</th>
																							<th></th>
																						</tr>
																					</thead>

																					<tbody>
																						<tr v-for="combo in item.cancelled_combos">
																							<!-- <td>{{ combo }} </td> -->
																							<!-- <td>
																								<input type="hidden" v-model="item.selected">
																							</td> -->
																							<td>{{ combo.product_type_name }}</td>
																							<td class="table-td">
																								{{ combo.quantity }}
																							</td>
																							<td>{{ combo.product_name }}</td>
																							<td width="75px">
																								{{ combo.stock_item.size ? combo.stock_item.size.name : '' }}
																							</td>
																							<td>
																								{{ combo.stock_item.color ? combo.stock_item.color.name : '' }}
																							</td>
																							<td class="table-td">
																								{{ combo.unit_price }}
																							</td>
																							<td>{{ (combo.quantity * combo.unit_price + (combo.quantity * combo.unit_price * (combo.tax.value/100))).toFixed(2) }}
																							</td>
																							<td>
																								<!-- Remove Button not available on cancelled document -->
																							</td>
																						</tr>
																					</tbody>
																				</table>
																			</div>
																		</div>
																	</div>

																</div>
															</td>
														</tr>
													</transition>
												</template>

												<template v-if="!cancelled" v-for="item in form.productsTableList">
													<tr @click="rowClicked(item, item.class)" :class="item.class">
														<td @click.stop class="cursor-pointer paddingless-side" align="center">
															<input type="hidden" v-model="item.selected">
															<span v-if="item.combos && item.combos.length"
																:class="'glyphicon glyphicon-chevron-' + (item.combos_show ? 'up' : 'down')"
																@click="toggleCombo(item)"
															>
															</span>
														</td>												
														<td>{{ item.type }}</td>
														<td class="table-td">
															<input @click.stop
																:disabled="closed || singletonQuantity(item) || isLocked"
																class="table-input-qty"
																type="text"
																v-model="item.quantity"
																@change="updateItem(item)"
															>
														</td>
														<td>{{ item.name }}</td>
														<td>{{ item.size }}</td>
														<td>{{ item.color }}</td>
														<td>{{ item.serial }}</td>
														<td>{{ item.location }}</td>
														<td class="table-td">
															<!-- Unit Price -->
															<input class="table-input"
																:disabled="closed || isLocked"
																type="text"
																@keypress="validate.isNumber"
																@change="updateItem(item)"
																@click.stop
																v-model="item.unit_price"
															>
														</td>
														<!-- <td>
															Discount
															<input type="text"
																class="table-input"
															>
														</td> -->
														<td>{{ (item.quantity * item.unit_price + (item.quantity * item.unit_price * (item.tax_value/100))).toFixed(2) }}</td>
														<td><a data-toggle="modal" data-target="#edit-shipping-modal" :id="item.related_shipping_id" data-backdrop="static" data-dismiss="modal" href="#">{{ item.related_shipping_document }}</a></td>
														<td><a data-toggle="modal" data-target="#edit-invoice-modal" :id="item.related_invoice_id" data-backdrop="static" data-dismiss="modal" href="#">{{ item.related_invoice_document }}</a></td>
														<td>
															<!-- <a v-if="!closed && item.type != 'Shipping' && !isLocked" @click.stop="removeItem(item)"> -->
															<a class="cursor-pointer" v-if="!closed && !isLocked && !disableRemoveButton(item)" @click.stop="removeItem(item)">
																<span class="glyphicon glyphicon-remove"></span>
															</a>
														</td>
													</tr>
													<transition name="combo">
														<tr v-if="item.combos && item.combos.length && item.combos_show" :class="item.class">
															<td colspan="12">
																<div class="row">
																	<div class="col-md-6 col-md-offset-1">
																		<div class="panel panel-primary panel-paddingless combos-table">
																			<div class="panel-heading" style="height: 20px;padding:0">
																				{{item.name}} ({{ item.serial }}) Accessories
																				<span v-if="!closed && !isLocked && !item.related_shipping_id && !item.related_invoice_id"
																					@click="showEditCombos(item)"
																					:class="'pull-right cursor-pointer p-r-10 glyphicon glyphicon-' + (item.show_edit_combos ? 'minus' : 'plus')"
																				>
																				</span>
																			</div>
																			<div class="panel-body">
																				<table class="table table-striped table-condensed new-proforma-table">
																					<thead>
																						<tr>
																							<!-- <th>
																								<input :disabled="form.productsTableList.length == 0" id="selectAllCheckbox" type="checkbox" @click="selectAllItems" v-model="selectAll">
																							</th> -->
																							<th>Item</th>
																							<th>Qty</th>
																							<th>Model</th>
																							<th>Size</th>
																							<th>Color</th>
																							<th>Unit Price</th>
																							<th>Total</th>
																							<th></th>
																						</tr>
																					</thead>

																					<tbody>
																						<tr v-for="combo in item.combos">
																							<!-- <td>{{ combo }} </td> -->
																							<!-- <td>
																								<input type="hidden" v-model="item.selected">
																							</td> -->												
																							<td>{{ combo.product_type_name }}</td>
																							<td class="table-td">
																								<input @click.stop
																									:disabled="singletonQuantity(item) || isLocked" 
																									class="table-input-qty"
																									type="text"
																									v-model="combo.quantity"
																								>
																							</td>
																							<td>{{ combo.product_name }}</td>
																							<td width="75px">
																								<!-- <span>{{ combo.stock_item.size ? combo.stock_item.size.name : '' }}</span> -->
																								<select @change="editCombo(item, combo)"
																									class="form-control"
																									v-model="combo.stock_item.size_id"
																									:disabled="closed || isLocked || item.related_shipping_id || item.related_invoice_id"
																								>
																									<option v-for="combo_size in combo.stock_item.product.sizes_from_stock"
																										:value="combo_size.id"
																									>
																										{{ combo_size.name }}
																									</option>
																								</select>
																							</td>
																							<td>
																								<!-- <span>{{ combo.stock_item.color ? combo.stock_item.color.name : '' }}</span> -->
																								<select @change="editCombo(item, combo)"
																									class="form-control"
																									v-model="combo.stock_item.color_id"
																									:disabled="closed || isLocked || item.related_shipping_id || item.related_invoice_id"
																								>
																									<option v-for="combo_color in combo.stock_item.product.colors_from_stock"
																										:value="combo_color.id"
																									>
																										{{ combo_color.name }}
																									</option>
																								</select>
																							</td>
																							<td class="table-td">
																								<input class="table-input"
																									type="text"
																									@click.stop
																									v-model="combo.unit_price"
																									@keypress="validate.isNumber"
																									@change="updateItem(combo, item)"
																									:disabled="closed || isLocked"
																								>
																							</td>
																							<td>{{ (combo.quantity * combo.unit_price + (combo.quantity * combo.unit_price * (combo.tax.value/100))).toFixed(2) }}
																							</td>
																							<td>
																								<a v-if="!closed && !isLocked && !item.related_shipping_id && !item.related_invoice_id"
																									@click="removeCombo(item, combo)"
																									class="cursor-pointer">
																									<span class="glyphicon glyphicon-remove"></span>
																								</a>
																							</td>
																						</tr>
																					</tbody>
																				</table>
																			</div>
																		</div>
																	</div>

																	<div v-if="item.show_edit_combos" class="col-md-3">
																		<table>
																			<tbody>
																				<tr v-for="cmb in item.combos_available">
																					<td><label class="label label-success">{{ cmb.name }}</label></td>
																					<td>
																						<select class="form-control" v-model="cmb.size">
																							<option disabled selected value="">Choose Size</option>
																							<option v-for="cmb_size in cmb.sizes_from_stock" :value="cmb_size.id">{{ cmb_size.name }}</option>
																						</select>
																					</td>
																					<td>
																						<select class="form-control" v-model="cmb.color">
																							<option disabled selected value="">Choose Color</option>
																							<option v-for="cmb_color in cmb.colors_from_stock" :value="cmb_color.id">{{ cmb_color.name }}</option>
																						</select>
																					</td>
																					<td>
																						<button @click="addCombo(item, cmb)"
																							type="button"
																							class="btn btn-default btn-sm"
																							:disabled="(cmb.sizes_from_stock.length > 0 && !cmb.size) || (cmb.colors_from_stock.length > 0 && !cmb.color)"
																						>
																							<span class="glyphicon glyphicon-plus"></span>
																						</button>
																					</td>
																				</tr>
																			</tbody>
																		</table>
																	</div>

																</div>
															</td>
														</tr>	
													</transition>
												</template>
											</tbody>
										</table>
									</div>
								</div>									
							</slot>
						</div>					
					</form>
					<div class="modal-footer">
						<slot name="footer">
							<!-- Print Buttons -->
							<button type="button" class="btn btn-default" @click="printWithoutCombos">
								<span class="fa fa-print"></span> Print without Accessories
							</button>
							<button type="button" class="btn btn-default" @click="print">
								<span class="fa fa-print"></span> Print
							</button>

							<!-- Back Button -->
							<button type="button" class="btn btn-default" data-dismiss="modal">Back</button>
							
							<!-- Save Button -->
							<button
								:disabled="!form.changes.any() || this.closed || form.errors.any()"
								type="button"
								class="btn btn-default"
								@click="onSubmit">
									Save
							</button>
							<!-- <button
								:disabled="!form.changes.any() || this.closed || form.errors.any()"
								type="button"
								class="btn btn-default"
								@click="onSubmitAndClose">
									Save &amp; Close
							</button> -->
						</slot>
					</div>	
				</div>
			</div>
		</div> <!-- /modal -->

		<confirm-dialog
			modal_id="cancel-proforma-confirm-modal"
			@confirm="statusConfirmation"
			title="Cancell Proforma Document"
			body="You will not be able to recover this action!"
			title_color="white"
			header_color="tomato"
		>
		</confirm-dialog>
	</div>
</template>

<script>
	export default {
		props: [ 'modal_id', 'proforma_id' ],

		created() {
			Event.$on('proforma-item-modal-hidden', () => { this.product_type = null; });
			Event.$on('selected-items-to-old-proforma', (items) => this.addSearialNumberItemsToTable(items));

			this.fetchProductTypeOptions();
		},

		mounted() {
			$.fn.modal.Constructor.prototype.enforceFocus = function() {};
			$(this.$refs.editproformamodal).on('shown.bs.modal', this.modalShown);
			$(this.$refs.editproformamodal).on("hidden.bs.modal", this.modalHidden);

			this.fetchStatusOptions();
			this.fetchDealerOptions();
			this.fetchShippingFromOptions();
			this.fetchTaxes();
			this.initDatepicker();
		},

		destroyed() {
			Event.$off('proforma-item-modal-hidden');
			Event.$off('selected-items-to-old-proforma');
		},

		data() {
			return {
				validate: new Validate(),
				closed: false,
				cancelled: false,
				reservableItem: true,
				default_price: 0,
				default_quantity: 1,
				proforma_status_options: null,
				proforma_dealer_options: null,
				proforma_shipping_from_options: null,
				form: new Form({
					proforma_id: '',
					proforma_user: '',
					proforma_name: '',
					date_created: '',
					document_date: '',
					proforma_status: null,
					proforma_dealer: null,
					shipping_from: null,
					intern_comment: '',
					comment: '',
					user_field: '',
					shipping_address: '',
					shipping_address_overwritten: '',
					invoice_address: '',
					invoice_address_overwritten: '',
					productsTableList: [],
					tax: null,
					cancelled_items: []
				}),
				taxes_options: [],
				open_shipping: null,
				open_shippings_options: null,
				product_type_options: null,
				product_model_options: null,
				product_color_options: null,
				product_size_options: null,
				product_type: null,
				product_type_name: null,
				product_size: null,
				product_size_name: null,
				product_color: null,
				product_color_name: null,
				product_model: '',
				product_model_name: '',

				selectAll: false,

				item_size_disabled: true,
				item_color_disabled: true,
				// dealer_field_disabled: true,
				edit_status: null,
				edit_message: ''
			}
		},

		computed: {
			canOpenOwnProformas() {
				if (Laravel.permissions.can_open_own_proformas) {
					if (parseInt(Laravel.userId) === parseInt(this.form.proforma_user)) {
						return true;
					}

					return false;
				}
			},

			canOpenAllProformas() {
				return Laravel.permissions.can_open_all_proformas ? true : false;
			},

			isOpen() {
				return this.form.proforma_status == 27 ? true : false;
			},

			isLocked() {
				return !this.edit_status;
			},

			disableStatusField() {
				if (this.isOpen && !this.isLocked) {
					return false;
				}
				else {
					console.log
					if (this.canOpenOwnProformas || this.canOpenAllProformas && !this.isLocked) {
						return false;
					}

					return true;
				}
			},

			disableAddToShippingBtn() {
				return (!this.checkIfSomethingSelected() || !Laravel.permissions.can_create_shippings);
				// return (!this.checkIfShippingSelected() || !Laravel.permissions.can_create_shippings);
			},

			disableDealerField() {
				return this.form.productsTableList.length == 0 ? false : true;
			},

			// disableRemoveButton() {
			// 	let shippings = this.form.productsTableList.filter( (element) => {
			// 		return element.product_id == this.form.shipping_from;
			// 	});

			// 	return shippings.length > 1 ? false : true;

			// 	item.type != 'Shipping'
			// },

			new_product_total() {
				return this.default_quantity * this.default_price;
			},

			shipping_address_title() {
				return this.form.shipping_address_overwritten ? 'Overwritten Dealer delivery address' : 'Dealer delivery address';
			},

			invoice_address_title() {
				return this.form.invoice_address_overwritten ? 'Overwritten Dealer invoice address' : 'Dealer invoice address';
			}
		},		

		methods: {
			modalShown(e) {
				let proforma_id = e.relatedTarget.id;
				this.form.proforma_id = proforma_id;
				this.fetchProformaData();
				this.lockProforma(proforma_id);			
			},

			modalHidden() {
				this.form.proforma_user = '';
				this.form.proforma_name = '';
				this.form.date_created = '';
				this.form.document_date = '';
				this.form.proforma_status = null;
				this.form.proforma_dealer = null;
				this.form.intern_comment = '';
				this.form.comment = '';
				this.form.user_field = '';
				this.form.shipping_address = '';
				this.form.invoice_address = '';
				this.form.shipping_from = null;
				this.form.tax = null;

				this.form.cancelled_items = [];
				this.form.productsTableList = [];

				this.product_type = null;
				this.reservableItem = true;
				this.selectAll = false;
				this.open_shipping = null;

			 	let checkboxAll = document.getElementById("selectAllCheckbox");

				checkboxAll.checked = false;
				checkboxAll.indeterminate = false;

				this.form.errors.clear();
				this.form.changes.clear();

				// Event.$emit('proforma-edit-modal-hidden');
				if (Proforma.vm) {
					Proforma.vm.$data.datatable.ajax.reload(null, false);
				}

				this.unlockProforma(this.form.proforma_id);
			},

			/**
			 * Update Proforma - Add Serial Number type item.
			 * 
			 * @param array items
			 */
			addSearialNumberItemsToTable (items) {
				let new_items = [];
				let errors = {};

				items.forEach((item) => {
					let new_item = {};

					new_item.type_id = item.product_type_id;
					new_item.type = item.product_type;
					new_item.name = item.name;
					new_item.color = item.color;
					new_item.size = item.size;
					new_item.unit_price = 0;
					new_item.quantity = 1;
					new_item.tax = this.form.tax;
					new_item.selected = false;
					new_item.stock_id = item.stock_id;
					new_item.serial = item.serial;
					new_item.combo_items = [];

					if (item.combos.length) {
						item.combos.forEach( (element) => {
							let combo_item = {};

							combo_item.product_type_id = element.product_type_id;
							combo_item.product_type = element.product_type;
							combo_item.product_id = element.id;
							combo_item.name = element.name;
							combo_item.color_id = element.color_id;
							combo_item.color = element.color;
							combo_item.size_id = element.size_id;
							combo_item.size = element.size;
							combo_item.unit_price = 0;					
							combo_item.quantity = 1;
							combo_item.selected = false;
							combo_item.stock_id = element.stock_id;
							combo_item.serial = element.serial;

							new_item.combo_items.push(combo_item);
						})
					}

					new_items.push(new_item);
				});

				if (!this.form.shipping_from) {
					errors["shipping_from"] = ["Shipping From field is required!"];
					this.form.errors.record(errors);
				}

				if (!this.form.errors.any()) {
					axios.patch('/documents/proforma/' + this.form.proforma_id, new_items)
						.then(response => {
							console.log(response.data);
							this.fetchProformaData();
							toastr.success(response.data.message);
						});
				}
			},

			addQuantityItemToTable() {
				let errors = {};
				// let without_warnings = true;

				if (!this.product_model) {
					errors["model"] = ["Model field is required!"];
					// toastr.warning('Model field is required!', 'Warning!');
					// without_warnings = false;
				}
				if (!this.item_color_disabled && !this.product_color) {
					errors["color"] = ["Color field is required!"];
					// toastr.warning('Color field is required!', 'Warning!');
					// without_warnings = false;
				}

				if (!this.item_size_disabled && !this.product_size) {
					errors["size"] = ["Size field is required!"];
					// toastr.warning('Size field is required!', 'Warning!');
					// without_warnings = false;
				}

				if (!this.form.shipping_from) {
					errors["shipping_from"] = ["Shipping From field is required!"];
					// without_warnings = false;
				}

				this.form.errors.record(errors);

				if (this.form.errors.any()) {
					for (var key in this.form.errors.errors) {
					    if (this.form.errors.errors.hasOwnProperty(key)) {
					        toastr.warning(this.form.errors.errors[key]);
					    }
					}
				}
				else {
					let new_item = [];
					let item = {};

					// Get Selected Product StockID
					axios.get('/api/common/product-stock-id', {
						params: {
							product_id: this.product_model,
							color_id: this.product_color,
							color: this.product_color_name,
							size_id: this.product_size,
							size: this.product_size_name
						}
					})
					.then(response => {
						console.log(response.data);
						let result = response.data;

						item.name = this.product_model_name;
						item.color_id = result.color_id;
						item.color = this.product_color_name;
						item.size_id = result.size_id;
						item.size = this.product_size_name;
						item.unit_price = this.default_price;
						item.type_id = this.product_type;
						item.type = this.product_type_name;
						item.quantity = this.default_quantity;
						item.tax = this.form.tax;
						item.selected = false;
						item.stock_id = result.id;
						item.serial = null;
						item.class = (result.related_shipping_document || !this.closed) ? 'active' : '';
						item.combo_items = [];

						this.form.productsTableList.push(item);
						
						this.product_model = '';
						this.product_size_options = null;
						this.product_color_options = null;
						this.default_price = 0;
						this.default_quantity = 1;
						this.product_color = null;
						this.product_color_name = null;
						this.product_size = null;
						this.product_size_name = null;

						new_item.push(item);

						axios.patch('/documents/proforma/' + this.form.proforma_id, new_item)
							.then(response => {
								console.log(response.data);
								this.fetchProformaData();
								toastr.success(response.data.message);
							});
					});
				}
			},

			addItemsToShippingDocument() {
				if (!this.open_shipping) {
					/*
						Create New Shipping Document
					 */
					
					let shipping_data = Object.assign({}, this.form);
					shipping_data.productsTableList = this.getSelectedItems();

					axios.post('/documents/shipping', shipping_data)
						.then(response => {
							console.log(response.data);
							this.fetchProformaData();
							toastr.success(response.data.message);
							this.open_shipping = null;
						})
				}
				else {
					/*
						Update Existing Shipping Document
					 */
					// let selected_items = this.getSelectedItems().filter( (obj) => {
					// 	return obj.type != 'Shipping';
					// });

					let selected_items = this.getSelectedItems();

					axios.patch('/documents/shipping/' + this.open_shipping, selected_items)
						.then(response => {
							console.log(response.data);
							this.fetchProformaData();
							toastr.success(response.data.message);
							this.open_shipping = null;
						});
				}
			},

			updateItem(item, parent_item = null) {
				// console.log(item)
				item.update_item = true;

				axios.patch('/documents/proforma/' + this.form.proforma_id, item)
					.then(response => {
						console.log(response.data);
						if (parent_item) {
							this.fetchProformaData(parent_item);
						} else {
							this.fetchProformaData();
						}
						toastr.success(response.data.message);
					});
			},

			/**
			 * Proforma Table Row is clicked (toggle row item selected class)
			 * 
			 * @param  Object item
			 * 
			 * @return void
			 */
			rowClicked(item, item_class) {
				if (this.closed && !item.related_shipping_id && item_class != 'active' && !this.isLocked) {
					var checkboxAll;

					item.selected = !item.selected;
					item.selected ? item.class = 'success' : item.class = '';
					
					if (this.checkIfSomethingSelected()) {
					 	checkboxAll = document.getElementById("selectAllCheckbox");
						checkboxAll.indeterminate = true;
					}
					else {
					 	checkboxAll = document.getElementById("selectAllCheckbox");
					 	checkboxAll.indeterminate = false;			
					}
				}
			},

			/**
			 * Select All Proforma Table Rows
			 * 
			 * @return void
			 */
			selectAllItems() {
			 	var checkboxAll = document.getElementById("selectAllCheckbox");

				if (this.checkIfSomethingSelected()) {
					checkboxAll.checked = false;
					checkboxAll.indeterminate = false;

					this.selectAll = false;
					this.form.productsTableList.forEach((element) => {
						if (element.class != 'active') {
							element.selected = false;
							element.class = '';
						}
					});					
				}
				else {
					this.form.productsTableList.forEach((element) => {
						if (element.class != 'active') {
							element.selected = !element.selected;
							element.selected ? element.class = 'success' : element.class = '';
						}
					});
				}
			},

			/**
			 * Check if some item from the proforma items table is selected
			 * 
			 * @return Boolean
			 */
			checkIfSomethingSelected() {
				for (var i=0; i < this.form.productsTableList.length; i++) {
					if (this.form.productsTableList[i].selected) {
						return true;
					}
				}

				return false;
			},

			/**
			 * Check if Shipping From [Country] item is selected
			 * 
			 * @return Boolean
			 */
			checkIfShippingSelected() {
				let shippingExists = this.form.productsTableList.filter( (element) => {
					return element.type == 'Shipping'; // && !element.related_shipping_id;
				});

				if (shippingExists.length) {
					// return true;
					let shipping_selected = this.form.productsTableList.filter( (element) => {
						return element.type == 'Shipping' && element.selected == true;
					});

					return shipping_selected.length > 0 ? true : false;
				}

				return this.checkIfSomethingSelected();
			},

			/**
			 * Check if Proforma has related documents
			 * 
			 * @return Boolean
			 */
			hasRelatedDocument() {
				for (var i=0; i < this.form.productsTableList.length; i++) {
					if (this.form.productsTableList[i].related_shipping_id) {
						return true;
					}
				}

				return false;
			},

			/**
			 * Disable input if Proforma is Closed, or if item is on other related Document.
			 * 
			 * @param Object item
			 * 
			 * @return Boolean
			 */
			disableInputs(item) {
				if (this.closed || item.related_shipping_id || item.related_invoice_id) {
					return true;
				}

				return false;
			},

			disableRemoveButton(item) {
				// let shippings = this.form.productsTableList.filter( (element) => {
				// 	return element.product_id == this.form.shipping_from;
				// });

				// if (item.type == 'Shipping' && shippings.length == 1 && this.form.productsTableList.length > 1)
				// 	return true;

				return false;
			},

			/**
			 * Disable input if Quantity is restricted to one.
			 * 
			 * @param  Object item
			 * 
			 * @return Boolean
			 */
			singletonQuantity(item) {
				if (item.type == 'Paraglider' || item.type == 'Harness' || item.type == 'Rescue' || item.type == 'Shipping') {
					return true;
				}

				return false;
			},

			/**
			 * Remove item from new profomra table
			 * 
			 * @param  Integer item_id
			 * @return Object productsTableList without removed item
			 */
			removeItem(item) {
				if (item.related_shipping_id || item.related_invoice_id) {
					// console.log("wa");
					toastr.warning("Item has related documents. Can't be removed!");
				}
				else {
					// let removed_item = this.form.productsTableList.find( (obj) => {
					// 	return obj.id == item.id;
					// });

					// this.form.productsTableList.splice(this.form.productsTableList.indexOf(removed_item), 1);


					axios.delete('/documents/proforma/' + this.form.proforma_id, {
						params: {
							'item': item.id
						}					
					})
					.then(response => {
						if (!response.data.success) {
							toastr.warning(response.data.message, 'Warning!');
							return;
						}

						this.fetchProformaData();
						toastr.success(response.data.message, 'Success!');
						// console.log(response.data)
					})
					.catch(error => {
						toastr.warning(error.response.data.error, 'Warning!');
						console.log(error.response.data.error);					
					});
				}
			},

			getSelectedItems() {
				let items = this.form.productsTableList;
				let selected_items = [];

				selected_items = items.filter( (obj) => {
					return obj.selected == true; // || obj.type == 'Shipping';
				});

				return selected_items;
			},

			/**
			 * Submits the form and updates Proforma Document
			 * 
			 * @return Response
			 */
			onSubmit() {
				axios.patch('/documents/proforma/' + this.form.proforma_id, this.form)
					.then(response => {
						console.log(response.data);
						Proforma.vm.$data.datatable.ajax.reload(null, false);
						this.form.changes.clear();
						this.fetchProformaData();
						toastr.success(response.data.message);
					})
					.catch(error => {
						// console.log(error.data);
						this.form.onFail(error.response.data);
						toastr.warning(error.response.data.error, 'Warning!');
					});
			},

			onSubmitAndClose() {
				axios.patch('/documents/proforma/' + this.form.proforma_id, this.form)
					.then(response => {
						console.log(response.data);
						$(this.$refs.editproformamodal).modal('hide');
						Proforma.vm.$data.datatable.ajax.reload(null, false);
						this.form.changes.clear();
						this.fetchProformaData();
						toastr.success(response.data.message);
					})
					.catch(error => {
						// console.log(error.data);
						this.form.onFail(error.response.data);
						toastr.warning(error.response.data.error, 'Warning!');
					});
			},

			/**
			 * Fetch Profoma Data
			 * 
			 * @return void
			 */
			fetchProformaData (parent_item = null) {
				if (this.form.productsTableList.length > 0) {
					this.form.productsTableList = [];
				}

				if (this.form.cancelled_items.length > 0) {
					this.form.cancelled_items = [];
				}

				axios.get('/documents/proforma/' + this.form.proforma_id)
					.then(response => {
						// console.log(response.data);
						let proforma = response.data;
						
						this.closed = (proforma.status_id == 29) ? true : false;
						this.cancelled = (proforma.status_id == 32) ? true : false;
						this.form.proforma_user = proforma.user_id,
						this.form.proforma_id = proforma.id;
						this.form.proforma_name = proforma.document_name;
						this.form.date_created = moment(proforma.date_creation).format('DD/MM/YYYY');
						this.form.document_date = proforma.date_document ? moment(proforma.date_document).format('DD/MM/YYYY') : '';
						this.form.proforma_status = proforma.status_id;
						this.form.proforma_dealer = proforma.client_id;
						// this.form.shipping_from = proforma
						this.form.intern_comment = proforma.intern_comment;
						this.form.comment = proforma.comment;
						this.form.user_field = proforma.user_field;
						this.form.shipping_address = proforma.shipping_address ? proforma.shipping_address : '';
						this.form.shipping_address_overwritten = parseInt(proforma.shipping_address_overwritten) ? true : false;
						this.form.invoice_address = proforma.invoice_address ? proforma.invoice_address : '';
						this.form.invoice_address_overwritten = parseInt(proforma.invoice_address_overwritten) ? true : false;
						this.form.tax = proforma.tax_id;

						/*
							Set Trackable fields for changes
						 */
						this.form.changes.setCurrentData('document_date', moment(proforma.date_document).format('DD/MM/YYYY'));
						this.form.changes.setCurrentData('intern_comment', proforma.intern_comment);
						this.form.changes.setCurrentData('comment', proforma.comment);
						this.form.changes.setCurrentData('user_field', proforma.user_field);
						this.form.changes.setCurrentData('shipping_address', proforma.shipping_address);
						this.form.changes.setCurrentData('invoice_address', proforma.invoice_address);
						this.form.changes.setCurrentData('proforma_dealer', proforma.client_id);

						// this.setDocumentVat(proforma.client_id);

						/*
							Product List Items
						 */
						proforma.document_items.forEach((element) => {
							let item = {};
// console.log(element);
							item.id = element.id;
							item.type_id = element.product_type_id;
							item.type = element.product_type_name;
							item.name = element.product_name;
							item.color = element.stock_item.color ? element.stock_item.color.name : '';
							item.size = element.stock_item.size ? element.stock_item.size.name : '';
							item.unit_price = element.unit_price;
							item.quantity = element.quantity;
							item.tax_id = proforma.tax_id;
							item.tax_value = element.tax.value;
							item.selected = false;
							// item.class = (element.related_shipping_document || element.product_type_name == 'Shipping' || !this.closed) ? 'active' : '';
							item.class = (element.related_shipping_document || !this.closed) ? 'active' : '';
							item.stock_id = element.stock_id;
							item.product_id = element.stock_item.product_id;
							item.serial = element.stock_item.full_serial;
							item.location = element.stock_item.warehouse ? element.stock_item.warehouse.name : '';
							item.related_proforma_id = proforma.id;
							item.related_shipping_id = element.related_shipping ? element.related_shipping.id : null;
							item.related_invoice_id = element.related_invoice ? element.related_invoice.id : null;
							item.related_proforma_document = proforma.document_name;
							item.related_shipping_document = element.related_shipping ? element.related_shipping.name : null;
							item.related_invoice_document = element.related_invoice ? element.related_invoice.name: null;
							item.combos = element.combos;
							if (parent_item) {
								item.combos_show = (element.id == parent_item.id) ? true : false;
							} else {
								item.combos_show = false;
							}
							// item.combos_available = element.stock_item.product.combos;
							item.combos_available = element.stock_item.product.combos.filter( (combo_available, index) => {
								let already_there = false;
								item.combos.forEach((combo) => {
									// console.log(combo_available.id, combo.product_id);
									// return combo_available.id != combo.product_id;
									if (combo_available.id == combo.product_id) {
										already_there = true;
										// delete element.stock_item.product.combos[index];
									}
								});

								if (!already_there) {
									combo_available.size = '';
									combo_available.color = '';
									return combo_available;
								}
							});
							item.show_edit_combos = false;

							this.form.productsTableList.push(item);
						});

						/*
							Cancelled Items
						 */
						proforma.cancelled_items.forEach((element) => {
							let item = {};

							item.id = element.id;
							item.type_id = element.product_type_id;
							item.type = element.product_type_name;
							item.name = element.product_name;
							item.color = element.stock_item.color ? element.stock_item.color.name : '';
							item.size = element.stock_item.size ? element.stock_item.size.name : '';
							item.unit_price = element.unit_price;
							item.quantity = element.quantity;
							item.tax_id = proforma.tax_id;
							item.tax_value = element.tax.value;
							item.selected = false;
							// item.class = (element.related_shipping_document || element.product_type_name == 'Shipping' || !this.closed) ? 'active' : '';
							item.class = (element.related_shipping_document || !this.closed) ? 'active' : '';
							item.stock_id = element.stock_id;
							item.product_id = element.stock_item.product_id;
							item.serial = element.stock_item.full_serial;
							item.location = element.stock_item.warehouse ? element.stock_item.warehouse.name : '';
							item.related_proforma_id = proforma.id;
							item.related_shipping_id = element.related_shipping ? element.related_shipping.id : null;
							item.related_invoice_id = element.related_invoice ? element.related_invoice.id : null;
							item.related_proforma_document = proforma.document_name;
							item.related_shipping_document = element.related_shipping ? element.related_shipping.name : null;
							item.related_invoice_document = element.related_invoice ? element.related_invoice.name: null;
							item.cancelled_combos = element.cancelled_combos;
							if (parent_item) {
								item.combos_show = (element.id == parent_item.id) ? true : false;
							} else {
								item.combos_show = false;
							}
							// // item.combos_available = element.stock_item.product.combos;
							// item.combos_available = element.stock_item.product.combos.filter( (combo_available, index) => {
							// 	let already_there = false;
							// 	item.combos.forEach((combo) => {
							// 		// console.log(combo_available.id, combo.product_id);
							// 		// return combo_available.id != combo.product_id;
							// 		if (combo_available.id == combo.product_id) {
							// 			already_there = true;
							// 			// delete element.stock_item.product.combos[index];
							// 		}
							// 	});

							// 	if (!already_there) {
							// 		combo_available.size = '';
							// 		combo_available.color = '';
							// 		return combo_available;
							// 	}
							// });
							// item.show_edit_combos = false;

							this.form.cancelled_items.push(item);
						});

						this.setShippingFrom();
						this.fetchOpenShippingDocuments();

						this.selectAll = false;
						document.getElementById("selectAllCheckbox").indeterminate = false;
					});
			},

			/**
			 * Fetch Status Options
			 * 
			 * @return Response
			 */
			fetchStatusOptions() {
				axios.get('/api/common/statuses/3')
					.then(response => this.proforma_status_options = response.data);
			},

			/**
			 * Event before Status Change (Select2)
			 *
			 * @param  Event evt
			 *
			 * @return void
			 */
			statusBeforeChange(evt) {
				if (evt.params.args.data.text == 'Cancelled') {
					if (this.hasRelatedDocument()) {
						evt.preventDefault();
						toastr.warning("This document has related documents! Can't be cancelled!");
					}
					else {
						evt.preventDefault();
						Event.$emit('confirm-dialog', evt);

						$('#cancel-proforma-confirm-modal').modal('show');
					}
				}
			},

			statusConfirmation(response) {
				if (response.confirm) {
	                // get select2 instance
	                var Select2 = $('#proforma-status').data('select2');
	                // remove prevented flag
	                delete response.event.params.args.prevented;
	                // Call trigger on the observer with select2 instance as context
	                Select2.constructor.__super__.trigger.call(Select2, 'select', response.event.params.args);
				}
			},

			/**
			 * Fetch Dealer Options
			 * 
			 * @return Response
			 */
			fetchDealerOptions() {
				axios.get('/api/clients/dealers')
					.then(response => this.proforma_dealer_options = response.data);
			},

			/**
			 * Fetch Open Shipping Documents
			 * 
			 * @return Response
			 */
			fetchOpenShippingDocuments() {
				axios.get('/api/documents/proformas/open-shipping-documents/' + this.form.proforma_dealer)
					.then(response => {
						this.open_shippings_options = response.data;
					});
			},

			/**
			 * Fetch Shipping From Options
			 * 
			 * @return Response
			 */
			fetchShippingFromOptions() {
				axios.get('/api/settings/shipping-management/shippings')
					.then(response => this.proforma_shipping_from_options = response.data);
			},

			/**
			 * Fetch Selected Shipping From
			 * 
			 * @param {int} shipping
			 * 
			 * @return {void}
			 */
			fetchSelectedShippingFrom(shipping) {
				// console.log(shipping);
				let current_shipping = this.form.productsTableList.find( (obj) => {
					return obj.type == 'Shipping';
				});

				// this.form.productsTableList.splice(this.form.productsTableList.indexOf(current_shipping), 1);

				if (current_shipping) {
					axios.get('/api/documents/proformas/shipping-from/' + shipping)
						.then(response => {
							current_shipping.stock_id = response.data.id;
							current_shipping.name = response.data.product.name;

							this.updateItem(current_shipping);
						});
				}
			},

			addShippingFrom(shipping) {
				let items = [];
				let current_shipping = this.form.productsTableList.find( (obj) => {
					return obj.type == 'Shipping';
				});

				if (!current_shipping) {
					axios.get('/api/documents/proformas/shipping-from/' + shipping)
						.then(response => {
							let item = {};
							let result = response.data;

							item.type_id = result.product.product_type.id;
							item.type = result.product.product_type.label;
							item.quantity = this.default_quantity;
							item.tax = this.form.tax;
							item.name = result.product.name;
							item.unit_price = this.default_price;
							item.selected = false;
							item.stock_id = result.id;
							item.serial = null;

							// this.form.productsTableList.push(item);
							items.push(item);

							axios.patch('/documents/proforma/' + this.form.proforma_id, items)
								.then(response => {
									console.log(response.data);
									this.fetchProformaData();
								});
						});
				}
			},

			/**
			 * Initialize Datepicker element
			 * 
			 * @return void
			 */
			initDatepicker() {
				$('#edit-proforma-document-date').datepicker({
					autoclose: true,
					format: 'dd/mm/yyyy'
				})
				.on('changeDate', (e) => {
					this.form.document_date = moment(e.date).format('DD/MM/YYYY');
				});
			},

			/**
			 * Set Document Shipping From
			 *
			 * @return void
			 */
			setShippingFrom() {
				let shipping_from = this.form.productsTableList.find( (obj) => {
					return obj.type == 'Shipping';
				});

				if (shipping_from)
					this.form.shipping_from = shipping_from.product_id;
				else
					this.form.shipping_from = null;
			},

			/**
			 * Open Proforma Document
			 * 
			 * @return Response
			 */
			openProformaDocument() {
				axios.patch('/documents/proforma/' + this.form.proforma_id, this.form)
					.then(response => {
						toastr.success(response.data.message);
						this.fetchProformaData();
						// console.log(response.data)
					})
					.catch(error => {
						toastr.warning(error.response.data.error, 'Warning!');
						console.log(error.response.data.error);
					});
			},

			/**
			 * Close Proforma Document
			 * 
			 * @return Response
			 */
			closeProformaDocument() {
				axios.patch('/documents/proforma/' + this.form.proforma_id, this.form)
					.then(response => {
						this.fetchProformaData();
						toastr.success(response.data.message);
						console.log(response.data)
					})
					.catch(error => {
						toastr.warning(error.response.data.error, 'Warning!');
						console.log(error.response.data.error);
					});
			},

			/**
			 * Cancel Proforma Document
			 *
			 * @return Response
			 */
			cancelProformaDocument() {
				axios.patch('/documents/proforma/' + this.form.proforma_id, this.form)
					.then(response => {
						this.fetchProformaData();
						toastr.success(response.data.message);
						console.log(response.data);
					})
					.catch(error => {
						toastr.warning(error.response.data.error, 'Warning!');
						console.log(error.response.data.error);
					});
			},

			changeVatValue(tax) {
				axios.patch('/documents/proforma/' + this.form.proforma_id, {
					'update_taxes': true,
					'tax_id': tax	
				})
				.then(response => {
					this.fetchProformaData();
					toastr.success(response.data.message);
				});
			},

			fetchTaxes() {
				axios.get('/api/common/taxes')
					.then(response => {
						response.data.forEach( (element) => {
							let item = {};						
							item.id = element.id;
							item.name = element.label;
							this.taxes_options.push(item);
						});
					});
			},

			setDocumentVat(dealer) {
				axios.get('/api/settings/client-management/clients/' + dealer)
					.then(response => this.form.tax = response.data.tax_id);
			},
			
			/**
			 * Fetch Product Type Options
			 * 
			 * @return Response
			 */
			fetchProductTypeOptions() {
				axios.get('/api/common/product-types-documents')
					.then(response => this.product_type_options = response.data);
			},

			/*
				Show/Hide Item Combos
			 */
			toggleCombo(item) {
				item.combos_show ? item.combos_show = false : item.combos_show = true;
			},

			showEditCombos(item) {
				console.log(item);
				item.show_edit_combos = !item.show_edit_combos;
			},

			/*
				Add new combo to item
			 */
			addCombo(item, combo) {
				axios.patch('/documents/proforma/' + this.form.proforma_id, {
					'add_combo': true,
					'item': item,
					'combo': combo
				})
				.then(response => {
					// console.log(response.data);
					this.fetchProformaData(item);
					toastr.success(response.data.message);
				});
			},

			/*
				Edit combo attributes (size, color)
			 */
			editCombo(item, combo) {
				axios.patch('/documents/proforma/' + this.form.proforma_id, {
					'edit_combo': true,
					'combo': combo
				})
				.then(response => {
					// console.log(response.data);
					this.fetchProformaData(item);
					toastr.success(response.data.message);
				});
			},

			/*
				Remove combo from item
			 */
			removeCombo(item, combo) {
				axios.patch('/documents/proforma/' + this.form.proforma_id, {
					'remove_combo': true,
					'combo': combo
				})
				.then(response => {
					// console.log(response.data);
					this.fetchProformaData(item);
					toastr.success(response.data.message);
				});
			},

			/*
				Lock Proforma for Edit
			 */
			lockProforma(proforma_id) {
				axios.patch('/documents/proforma/' + proforma_id, {
					'lock': 1
				})
				.then(response => {
					console.log(response.data);
					this.edit_status = response.data.edit_status;
					this.edit_message = response.data.message;
					// this.fetchProformaData();
					// toastr.success(response.data.message);
				});
				// axios.post('/')
			},

			/*
				Unlock Proforma for Edit
			 */
			unlockProforma(proforma_id) {
				axios.patch('/documents/proforma/' + proforma_id, {
					'unlock': 1
				})
				.then(response => {
					console.log(response.data);
					this.edit_status = null;
					this.edit_message = '';
					// this.fetchProformaData();
					// toastr.success(response.data.message);
				});
			},

			/**
			 * Print Proforma Document
			 * 
			 * @return void
			 */
			print() {
				window.open('/documents/print/proforma/' + this.form.proforma_id + '/true', '_blank');
			},

			/**
			 * Print Proforma Document Without Combos
			 * 
			 * @return void
			 */
			printWithoutCombos() {
				window.open('/documents/print/proforma/' + this.form.proforma_id, '_blank');
			}
		},

		watch: {
			'form.proforma_status'(newVal, oldVal) {
				if (newVal && oldVal && newVal != oldVal) {
					if (newVal == 29) {
						/*
							If Proforma Status set to "Closed"
						 */
						this.closeProformaDocument();
					}

					if (newVal == 27) {
						/*
							If Proforma Status set to "Open"
						 */
						this.openProformaDocument();
					}

					if (newVal == 32) {
						/*
							If Proforma Status set to "Cancelled"
						 */
						if (this.hasRelatedDocument()) {
							toastr.warning("This document has related documents! Can't be cancelled!");
						}
						else {
							this.cancelProformaDocument();
						}
					}
				}					
			},

			'form.proforma_dealer'(newVal) {
				this.form.changes.check('proforma_dealer', newVal);
			},

			'form.document_date'(newVal) {
				this.form.changes.check('document_date', newVal);
			},

			'form.intern_comment'(newVal) {
				this.form.changes.check('intern_comment', newVal);
			},

			'form.comment'(newVal) {
				this.form.changes.check('comment', newVal);
			},

			'form.user_field'(newVal) {
				this.form.changes.check('user_field', newVal);
			},

			'form.shipping_address'(newVal) {
				this.form.changes.check('shipping_address', newVal);
			},

			'form.invoice_address'(newVal) {
				this.form.changes.check('invoice_address', newVal);
			},

			'form.shipping_from'(newVal, oldVal) {
				if (newVal && oldVal && newVal != oldVal) {
					this.product_model_options = this.proforma_shipping_from_options.filter( (element) => {
						return element.id == this.form.shipping_from;
					});

					this.fetchSelectedShippingFrom(newVal);
				}

				if (!oldVal && newVal) {
					this.addShippingFrom(newVal);
				}

				this.form.errors.clear('shipping_from');
			},

			'form.tax'(newVal, oldVal) {
				if (newVal && oldVal && newVal != oldVal) {
					this.changeVatValue(newVal);
				}
			},

			product_type(item) {
				// console.log(item)
				// Get Selected Product Type Name
				axios.get('/api/common/product-type/' + item)
					.then(response => {
						this.product_type_name = response.data.label;
					});

				var reservableItems = [1, 2, 4]; // (Paraglider, Rescue, Harness)

				if (reservableItems.includes(parseInt(item))) {
					/*
						Product Category is SERIAL NUMBER
					 */
				
					this.reservableItem = true;

					Event.$emit('proforma-new-item-modal', {
						proforma_id: this.form.proforma_id,
						item: item,
						dealer: this.form.proforma_dealer,
						productsTableList: this.form.productsTableList
					});
				}
				else if (!item) {
					// Item is not selected
					this.reservableItem = true;
				}
				else {
					/*
						Product Category is QUANTITY
					 */
					if (item == 6) {
						// Shipping Country
						this.product_model_options = this.proforma_shipping_from_options.filter( (element) => {
							return element.id == this.form.shipping_from;
						})

						if (this.product_model_options.length == 1) {
							this.product_model = this.product_model_options[0].id;
						}
					}
					else {
						// Item Model Options
						axios.get('/api/common/products/' + this.product_type)
							.then(response => this.product_model_options = response.data);
					}

					this.reservableItem = false;
				}
			},

			product_model(item) {
				if (item) {
					this.form.errors.clear('model');

					// Get Selected Product Name
					axios.get('/api/common/product-name/' + item)
						.then(response => {
							this.product_model_name = response.data.name;
						});

					// Get Product Colors
					// axios.get('/api/common/product-colors-from-stock/' + item)
					axios.get('/api/settings/product-management/attribute-values-stock/' + item + '/color')
						.then(response => {
							this.product_color_options = response.data;

							if (this.product_color_options.length == 1) {
								this.product_color = this.product_color_options[0].id;
							}
							else if (this.product_color_options.length > 1) {
								this.item_color_disabled = false;
							}
							else {
								this.item_color_disabled = true;
							}
						});

					// Get Product Sizes
					// axios.get('/api/common/product-sizes-from-stock/' + item)
					axios.get('/api/settings/product-management/attribute-values-stock/' + item + '/size')
						.then(response => {
							this.product_size_options = response.data;

							if (this.product_size_options.length == 1) {
								this.product_size = this.product_size_options[0].id;
							}
							else if (this.product_size_options.length > 1) {
								this.item_size_disabled = false;
							}
							else {
								this.item_size_disabled = true;
							}
						});
				}
			},

			product_color(item) {
				axios.get('/api/common/attribute-value/' + item)
					.then(response => {
						this.product_color_name = response.data.name;
					});

				this.form.errors.clear('color');
			},

			product_size(item) {
				axios.get('/api/common/attribute-value/' + item)
					.then(response => {
						this.product_size_name = response.data.name;
					});

				this.form.errors.clear('size');
			}			
		}
	}
</script>

<style scoped>
	#add-to-shipping-btn {
		line-height: 1;
		border-radius: 4px;
		margin-right: 4px;
	}
	.combo-enter-active, .combo-leave-active {
	  transition: opacity .5s;
	}
	.combo-enter, .combo-leave-to /* .fade-leave-active below version 2.1.8 */ {
	  opacity: 0;
	}
	.item-price, .item-total {
		width: 150px;
	}
</style>