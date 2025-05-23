<template>
    <Combobox by="id" v-model="selectedService">
        <ComboboxAnchor>
            <div class="relative w-full max-w-sm items-center">
                <ComboboxInput
                    class="pl-9"
                    :display-value="(val) => val?.name ?? ''"
                    :model-value="texto"
                    placeholder="Seleccionar servicio..."
                    @update:model-value="handleSearchInput"
                />
                <span class="absolute inset-y-0 start-0 flex items-center justify-center px-3">
                    <Search class="size-4 text-muted-foreground" />
                </span>
                <!-- search -->
                <span v-if="isSearching" class="absolute inset-y-0 end-0 flex items-center justify-center px-3">
                    <div class="h-4 w-4 animate-spin rounded-full border-b-2 border-t-2 border-primary"></div>
                </span>
            </div>
        </ComboboxAnchor>
        <ComboboxList>
            <ComboboxEmpty>No se encontró ningún servicio.</ComboboxEmpty>
            <ComboboxGroup>
                <ComboboxItem v-for="service in services" :key="service.id" :value="service" @select="onSelect(service)">
                    {{ service.name }}
                    <ComboboxItemIndicator>
                        <Check class="ml-auto h-4 w-4" />
                    </ComboboxItemIndicator>
                </ComboboxItem>
            </ComboboxGroup>
        </ComboboxList>
    </Combobox>
</template>
<script setup lang="ts">
import { usePayment } from '@/composables/usePayment';
import { InputService } from '@/interface/Inputs';
import { Check, Search } from 'lucide-vue-next';
import { onMounted, ref, watch } from 'vue';
import {
    Combobox,
    ComboboxAnchor,
    ComboboxEmpty,
    ComboboxGroup,
    ComboboxInput,
    ComboboxItem,
    ComboboxItemIndicator,
    ComboboxList,
} from '../ui/combobox';

const emit = defineEmits<{
    (e: 'select', service_id: number): void;
}>();

const { services, getServices } = usePayment();
const texto = ref<string>('');
const isSearching = ref<boolean>(false);
const selectedService = ref<InputService | null>(null);

const handleSearchInput = (value: string) => {
    texto.value = value.toLowerCase();
};

const onSelect = (service: InputService) => {
    selectedService.value = service;
    texto.value = service.name;
    emit('select', service.id);
};

const debounceSearch = async () => {
    try {
        isSearching.value = true;
        await getServices(texto.value);
    } catch (e) {
        console.error(e);
    } finally {
        isSearching.value = false;
    }
};

watch(
    texto,
    (newValue) => {
        if (newValue.length > 2) {
            debounceSearch();
        }
    },
    { immediate: true },
);

onMounted(() => {
    debounceSearch();
});
</script>
<style scoped></style>
