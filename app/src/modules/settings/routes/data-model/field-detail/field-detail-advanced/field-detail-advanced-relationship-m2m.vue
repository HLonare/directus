<template>
	<div>
		<div class="grid">
			<div class="field">
				<div class="type-label">{{ t('this_collection') }}</div>
				<v-input disabled :model-value="collection" />
			</div>

			<div class="field">
				<div class="type-label">{{ t('junction_collection') }}</div>

				<related-collection-select
					v-model="junctionCollection"
					:disabled="autoGenerateJunctionRelation || isExisting"
				/>
			</div>

			<div class="field">
				<div class="type-label">{{ t('related_collection') }}</div>
				<related-collection-select v-model="relatedCollection" :disabled="type === 'files' || isExisting" />
			</div>

			<v-input disabled :model-value="currentPrimaryKey" />

			<related-field-select
				v-model="junctionFieldCurrent"
				:collection="junctionCollection"
				:disabled="autoGenerateJunctionRelation || isExisting"
			/>

			<div class="spacer" />
			<div class="spacer" />

			<related-field-select
				v-model="junctionFieldRelated"
				:collection="junctionCollection"
				:disabled="autoGenerateJunctionRelation || isExisting"
			/>

			<v-input v-model="relatedPrimaryKey" disabled :placeholder="t('primary_key') + '...'" />

			<div class="spacer" />

			<v-checkbox v-if="!isExisting" v-model="autoGenerateJunctionRelation" block :label="t('auto_fill')" />

			<v-icon class="arrow" name="arrow_forward" />
			<v-icon class="arrow" name="arrow_backward" />
		</div>

		<v-divider v-if="!isExisting" large :inline-title="false">{{ t('corresponding_field') }}</v-divider>

		<div v-if="!isExisting" class="corresponding">
			<div class="field">
				<div class="type-label">{{ t('create_field') }}</div>
				<v-checkbox v-model="hasCorresponding" block :label="correspondingLabel" />
			</div>
			<div class="field">
				<div class="type-label">{{ t('field_name') }}</div>
				<v-input
					v-model="correspondingFieldKey"
					:disabled="hasCorresponding === false"
					:placeholder="t('field_name') + '...'"
					db-safe
				/>
			</div>
			<v-icon name="arrow_forward" class="arrow" />
		</div>

		<div class="sort-field">
			<v-divider large :inline-title="false">{{ t('sort_field') }}</v-divider>
			<related-field-select
				v-model="sortField"
				:collection="junctionCollection"
				:placeholder="t('add_sort_field') + '...'"
			/>
		</div>

		<div class="relational-triggers">
			<v-divider class="field full" large :inline-title="false">{{ t('relational_triggers') }}</v-divider>

			<div class="field">
				<div class="type-label">
					{{
						t('referential_action_field_label_o2m', {
							collection: junctionCollection || '',
						})
					}}
				</div>
				<v-select
					v-model="deselectAction"
					:placeholder="t('choose_action') + '...'"
					:items="[
						{
							text: t('referential_action_set_null', {
								collection: junctionCollection,
								field: junctionFieldCurrent,
							}),
							value: 'nullify',
						},
						{
							text: t('referential_action_cascade', {
								collection: junctionCollection,
								field: junctionFieldCurrent,
							}),
							value: 'delete',
						},
					]"
				/>
			</div>

			<div class="field">
				<div class="type-label">
					{{
						t('referential_action_field_label_m2o', {
							collection: collection || 'related',
						})
					}}
				</div>
				<v-select
					v-model="onDeleteCurrent"
					:disabled="junctionCollection === collection"
					:placeholder="t('choose_action') + '...'"
					:items="[
						{
							text: t('referential_action_set_null', { field: junctionFieldCurrent }),
							value: 'SET NULL',
						},
						{
							text: t('referential_action_set_default', { field: junctionFieldCurrent }),
							value: 'SET DEFAULT',
						},
						{
							text: t('referential_action_cascade', {
								collection: junctionCollection,
								field: junctionFieldCurrent,
							}),
							value: 'CASCADE',
						},
						{
							text: t('referential_action_no_action'),
							value: 'NO ACTION',
						},
					]"
				/>
			</div>

			<div class="field">
				<div class="type-label">
					{{
						t('referential_action_field_label_m2o', {
							collection: relatedCollection || 'related',
						})
					}}
				</div>
				<v-select
					v-model="onDeleteRelated"
					:disabled="junctionCollection === relatedCollection"
					:placeholder="t('choose_action') + '...'"
					:items="[
						{
							text: t('referential_action_set_null', { field: junctionFieldRelated }),
							value: 'SET NULL',
						},
						{
							text: t('referential_action_set_default', { field: junctionFieldRelated }),
							value: 'SET DEFAULT',
						},
						{
							text: t('referential_action_cascade', {
								collection: junctionCollection,
								field: junctionFieldRelated,
							}),
							value: 'CASCADE',
						},
						{
							text: t('referential_action_no_action'),
							value: 'NO ACTION',
						},
					]"
				/>
			</div>
		</div>

		<v-notice v-if="generationInfo.length > 0" class="generated-data" type="warning">
			<span>
				{{ t('new_data_alert') }}

				<ul>
					<li v-for="(data, index) in generationInfo" :key="index">
						<span class="field-name">{{ data.name }}</span>
					</li>
				</ul>
			</span>
		</v-notice>
	</div>
</template>

<script lang="ts">
import { useI18n } from 'vue-i18n';
import { defineComponent, computed } from 'vue';
import { useFieldDetailStore, syncFieldDetailStoreProperty } from '../store';
import { storeToRefs } from 'pinia';
import RelatedCollectionSelect from '../shared/related-collection-select.vue';
import RelatedFieldSelect from '../shared/related-field-select.vue';
import { useFieldsStore } from '@/stores';

export default defineComponent({
	components: { RelatedCollectionSelect, RelatedFieldSelect },
	setup() {
		const { t } = useI18n();

		const fieldDetailStore = useFieldDetailStore();
		const fieldsStore = useFieldsStore();

		const { field, collection, editing, generationInfo } = storeToRefs(fieldDetailStore);

		const sortField = syncFieldDetailStoreProperty('relations.o2m.meta.sort_field');
		const junctionCollection = syncFieldDetailStoreProperty('relations.o2m.collection');
		const junctionFieldCurrent = syncFieldDetailStoreProperty('relations.o2m.field');
		const junctionFieldRelated = syncFieldDetailStoreProperty('relations.m2o.field');
		const relatedCollection = syncFieldDetailStoreProperty('relations.m2o.related_collection');
		const autoGenerateJunctionRelation = syncFieldDetailStoreProperty('autoGenerateJunctionRelation');
		const onDeleteCurrent = syncFieldDetailStoreProperty('relations.o2m.schema.on_delete');
		const onDeleteRelated = syncFieldDetailStoreProperty('relations.m2o.schema.on_delete');
		const deselectAction = syncFieldDetailStoreProperty('relations.o2m.meta.one_deselect_action');
		const correspondingField = syncFieldDetailStoreProperty('fields.corresponding');
		const correspondingFieldKey = syncFieldDetailStoreProperty('fields.corresponding.field');

		const type = computed(() => field.value.type);
		const isExisting = computed(() => editing.value !== '+');

		const currentPrimaryKey = computed(() => fieldsStore.getPrimaryKeyFieldForCollection(collection.value!)?.field);

		const relatedPrimaryKey = computed(
			() => fieldsStore.getPrimaryKeyFieldForCollection(relatedCollection.value)?.field ?? 'id'
		);

		const hasCorresponding = computed({
			get() {
				return !!correspondingField.value;
			},
			set(enabled: boolean) {
				if (enabled) {
					correspondingField.value = {
						field: collection.value,
						collection: relatedCollection.value,
						type: 'alias',
						meta: {
							special: ['m2m'],
							interface: 'list-m2m',
						},
					};
				} else {
					correspondingField.value = null;
				}
			},
		});

		const correspondingLabel = computed(() => {
			if (junctionCollection.value) {
				return t('add_m2m_to_collection', { collection: relatedCollection.value });
			}

			return t('add_field_related');
		});

		return {
			t,
			autoGenerateJunctionRelation,
			collection,
			type,
			isExisting,
			junctionCollection,
			junctionFieldCurrent,
			relatedCollection,
			sortField,
			currentPrimaryKey,
			junctionFieldRelated,
			relatedPrimaryKey,
			onDeleteCurrent,
			onDeleteRelated,
			deselectAction,
			hasCorresponding,
			correspondingLabel,
			correspondingFieldKey,
			generationInfo,
		};
	},
});
</script>

<style lang="scss" scoped>
@import '@/styles/mixins/form-grid';

.grid {
	--v-select-font-family: var(--family-monospace);
	--v-input-font-family: var(--family-monospace);

	position: relative;
	display: grid;
	grid-template-columns: repeat(3, minmax(0, 1fr));
	gap: 12px 28px;
	margin-top: 48px;

	.v-icon.arrow {
		--v-icon-color: var(--primary);

		position: absolute;
		transform: translateX(-50%);
		pointer-events: none;

		&:first-of-type {
			top: 117px;
			left: 32.5%;
		}

		&:last-of-type {
			top: 190px;
			left: 67.4%;
		}
	}
}

.v-input.matches {
	--v-input-color: var(--primary);
}

.type-label {
	margin-bottom: 8px;
}

.v-divider {
	margin: 48px 0 24px;
}

.v-list {
	--v-list-item-content-font-family: var(--family-monospace);
}

.corresponding {
	position: relative;
	display: grid;
	grid-template-columns: repeat(2, minmax(0, 1fr));
	gap: 12px 32px;

	.arrow {
		--v-icon-color: var(--primary);

		position: absolute;
		bottom: 17px;
		left: 50%;
		transform: translateX(-50%);
	}
}

.v-notice {
	margin-bottom: 36px;
}

.generated-data {
	margin-top: 36px;

	ul {
		padding-top: 4px;
		padding-left: 24px;
	}

	.field-name {
		font-family: var(--family-monospace);
	}
}

.sort-field {
	--v-input-font-family: var(--family-monospace);

	.v-divider {
		margin-top: 48px;
		margin-bottom: 24px;
	}
}

.relational-triggers {
	--form-horizontal-gap: 12px;
	--form-vertical-gap: 24px;

	@include form-grid;

	.v-divider {
		margin-top: 48px;
		margin-bottom: 0;
	}
}
</style>
