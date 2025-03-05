<template>
    <div class="select">
        <el-text class="mx-1" type="primary" size="large">当前格子数</el-text>
        <el-select v-model="forCount" placeholder="格子数量" size="large" style="width: 240px">
            <el-option :value=200 />
            <el-option :value=300 />
            <el-option :value=400 />
            <el-option :value=500 />
            <el-option :value=600 />
            <el-option :value=700 />
            <el-option :value=800 />
            <el-option :value=900 />
            <el-option :value=1000 />
        </el-select>
    </div>
    <div class="container">
        <div id="answerSheet">
            <div class="row" v-for="count in forCount">
                <el-input ref="inputRef" :model-value="answers[count]"
                    @update:model-value="(val: string) => handleInput(val, count)" :id="`input-${count}`"
                    @input="updateWordCount" class="character-input" @focus="handleFocus(count)"
                    @keydown="(e: KeyboardEvent) => handleKeyDown(e, count)"></el-input>
            </div>
        </div>
        <div id="counter" class="word-counter">字数：{{ wordCount }}</div>
        <div v-for="num in forCount / 100">
            <el-text class="number" type="danger" size="default"
                :style="{ top: `${(num * 100 * 50) / 20 + 25}px` }">{{ num }}00字</el-text>
        </div>
    </div>

</template>
<script lang="ts" setup>
import { onMounted, ref } from 'vue';

const forCount = ref(200);
const answers = ref<string[]>([]);
const wordCount = ref(0)
//现在的聚焦的下标
const nowIndex = ref(0)
const inputRef = ref(null)


const updateWordCount = () => {
    // 重新计数
    wordCount.value = answers.value.filter(item => /^[\u4e00-\u9fa5]+$/.test(item)).length
};
onMounted(() => {
    answers.value = Array(forCount.value).fill('')
});
const handleFocus = (count: number) => {
    nowIndex.value = count

}
const handleInput = (value: string, count: number) => {
    // 过滤非汉字字符
    // 允许处理空值（删除操作）
    if (value === answers.value[count]) return;

    // 当新值为空时直接更新（处理删除操作）
    if (!value) {
        answers.value[count] = '';
        updateWordCount();
        return;
    }
    // const chineseChars = value.replace(/[^\u4e00-\u9fa5]/g, '');
    const chineseChars = value
    // 将字符逐个填入后续输入框
    Array.from(chineseChars).forEach((char, index) => {
        const targetIndex = count + index;
        if (targetIndex < forCount.value) {
            answers.value[targetIndex] = char;
        }
    });

    // 自动聚焦到最后一个填充位置的下一个输入框
    const nextIndex = Math.min(count + chineseChars.length, forCount.value - 1);
    const nextInput = document.getElementById(`input-${nextIndex}`) as HTMLInputElement;
    nextInput?.focus();

    updateWordCount();
}
const handleKeyDown = (e: KeyboardEvent, count: number) => {

    if (e.key === 'Backspace' && answers.value[count] == '') {
        e.preventDefault()
        const prevIndex = Math.max(count - 1, 0)
        const prevInput = document.getElementById(`input-${prevIndex}`) as HTMLInputElement
        prevInput?.focus()
    }

}

</script>

<style scoped>
.number {
    position: absolute;
    right: -38px;
    transform: translateY(-50%);
}

.select {
    position: relative;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    margin: 10px 0;
    width: 100%;
    top: 18px;
}

.container {
    position: relative;
    width: 1006px;
    margin: 10px auto;
    padding-top: 25px;
    font-family: 'SimSun', Courier, monospace;
}

#answerSheet {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    border: 3px solid #409eff;
    
}

.word-counter {
    position: absolute;
    right: -80px;
    top: 0;
    color: #666;
}

.character-input {
    width: 50px;
    height: 50px;
    border: none;
    text-align: center;
    font-size: 16px;
}
</style>