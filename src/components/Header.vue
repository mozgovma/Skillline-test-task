<template>
    <div class="header">
        <div class="header-upper-part">
            <div class="header-logo">
                <img :src="MainLogo" alt="">
            </div>
            <div class="header-info">
                <img :src="Logo" alt="">
                <p>+7 962 253 20 37</p>
                <p>sales@skillline.ru</p>
                <a class="header-link" href="https://skillline.ru/">skillline.ru</a>
            </div>
        </div>
        <div class="header-title">
            <h1 class="title">Тестовое задание</h1>
        </div>
    </div>
</template>

<script setup>
import MainLogo from "@/assets/images/[icon].png"
import Logo from "@/assets/images/[logo]2.png"
</script>