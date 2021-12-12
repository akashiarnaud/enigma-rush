<template>
    <pre>{{ historique }}</pre>
</template>

<script>
export default {
    data: function(){
        return {
            historique : [],
        }
    },
    mounted() {
        this.historique = JSON.parse(localStorage.getItem("data"));
    }
}
</script>>