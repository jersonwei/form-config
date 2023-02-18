<template>
    <div class="resynchronize-container col">
        <span>当前关联的物模型已经失效，您可以执行以下操作：</span>
        <span>1、将 【设备来源】变更为：系统分类</span>
        <span>2、删除当前设备类型</span>
    </div>
</template>

<script>
export default {
    data() {
        return {}
    }
}
</script>

<style lang="scss" scoped>
.resynchronize-container {
    padding: 10px 0;

    span {
        line-height: 20px;
    }
}
</style>
