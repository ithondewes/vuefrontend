<script setup>
import { ref } from 'vue';

import AppMenuItem from './AppMenuItem.vue';

const model = ref([
    {
        label: 'Home',
        items: [{ label: 'Dashboard', icon: 'pi pi-fw pi-home', to: '/' }]
    },
    {
        items: [
            { label: 'Relatórios', icon: 'pi pi-book', to: '/uikit/table' },
        ]
    },
    {
        to: '/pages',
        items: [
            {
                label: 'Advisor',
                icon: 'pi pi-fw pi-user',
                items: [
                    {
                        label: 'Lista de clientes',
                        to: '/uikit/list'
                    },
                    {
                        label: 'Cobrança',
                        to: '/auth/error'
                    },
                    {
                        label: 'Estatísticas',
                        to: '/auth/access'
                    }
                ]
            }
        ]
    },
    {
        items: [
            {
                label: 'Mensagens',
                icon: 'pi pi-comment',
                to: '/documentation'
            }
        ]
    },
    {
        items: [
            {
                label: 'Ajuda',
                icon: 'pi pi-question-circle',
                to: '/icons'
            }
        ]
    }
]);
</script>

<template>
    <ul class="layout-menu">
        <template v-for="(item, i) in model" :key="item">
            <app-menu-item v-if="!item.separator" :item="item" :index="i"></app-menu-item>
            <li v-if="item.separator" class="menu-separator"></li>
        </template>
    </ul>
</template>

<style lang="scss" scoped></style>
