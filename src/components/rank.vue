<template>
<div class="rankBox">
    <h2>排行榜</h2>
    <div class="title">
        <span class="num">排名</span><span class="name">姓名</span><span class="time">时间</span>
    </div>
    <ul>
        <li>
            <span class="num">1</span><span class="name">name</span><span class="time">00:00</span>
        </li>
    </ul>
</div>
</template>
<script>

export default ({
  name: 'Rank',
  data () {
    return {

    }
  }
})
</script>
<style scoped>
.rankBox{
    width: 300px;
    /* background: #66CCCC; */
    margin-left: 100px;
}
h2{
    text-align: center;
}
ul{
    padding:0;
    margin:0;
}
li,.title{
    list-style: none;
    display: flex;
    justify-content: center;
}
.num{
    padding:0 10px;
    display: inline-block;
    min-width: 80px;
}
.name{
    display: inline-block;
    padding:0 10px;
    min-width: 100px;
}
</style>
