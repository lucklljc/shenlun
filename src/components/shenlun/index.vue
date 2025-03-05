<template>
    <div class="operate-area">
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
    <el-button type="primary" class="copy" @click="handleCopy">复制当前内容到粘贴板</el-button>
    <el-button type="primary" class="copy" @click="handleClear">清空当前内容</el-button>
    <div class="save">
        <div v-loading="loading" class="loading"></div>
        <el-text class="text-save" v-if="!loading" type="primary" >内容已保存!</el-text>
        <el-text class="text-save" v-else type="danger">保存中</el-text>
    </div>
    </div>
    
    
    <div class="container">
        <div id="answerSheet">
            <div class="row" v-for="count in forCount">
                <el-input ref="inputRef" :model-value="answers[count]"
                    max-length="1"
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

//TODO
//新增内容保存,存本地 ✔
//新增文件保存 
//新增日志记录->github
//新增导出内容->粘贴到粘贴板
//新增清空功能 ✔
import { onMounted, ref, watch } from 'vue';
import { ElMessage } from 'element-plus';
const forCount = ref(200);
const answers = ref<string[]>([]);
const wordCount = ref(0)
//现在的聚焦的下标
const nowIndex = ref(0)
const inputRef = ref(null)
const loading = ref(false)

const updateWordCount = () => {
    // 重新计数
    wordCount.value = answers.value.filter(item => /^[\u4e00-\u9fa5]+$/.test(item)).length
};
onMounted(() => {
    // answers.value = Array(forCount.value).fill('')
    const saved = localStorage.getItem('content')
    const isNull = saved ? 
        JSON.parse(saved)
            .filter((item: string) => item ? /^[\u4e00-\u9fa5]+$/.test(item) : false)
            .length === 0 
        : false
    // console.log(isNull);
    if(!isNull){
        ElMessage({
            type:'success',
            message:'内容恢复成功'
        })
    }
    answers.value = saved ? JSON.parse(saved) : Array(forCount.value).fill('')
    

});

const handleFocus = (count: number) => {
    nowIndex.value = count

}
const saveAnwser = () => {
    loading.value=true
    setTimeout(function(){
        localStorage.setItem('content',JSON.stringify(answers.value))
        loading.value=false
    },500)
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
    saveAnwser();
}

const handleKeyDown = (e: KeyboardEvent, count: number) => {

    if (e.key === 'Backspace' && answers.value[count] == '') {
        e.preventDefault()
        const prevIndex = Math.max(count - 1, 0)
        const prevInput = document.getElementById(`input-${prevIndex}`) as HTMLInputElement
        prevInput?.focus()
        saveAnwser()
    }
}

watch(forCount, (newVal) => {
    const current = answers.value
    answers.value = current.length > newVal 
        ? current.slice(0, newVal)
        : [...current, ...Array(newVal - current.length).fill('')]
    localStorage.setItem('content', JSON.stringify(answers.value))
})

const handleCopy = () => {
    const str = answers.value.filter(item => item != '').join('')
    console.log(str);
    navigator.clipboard.writeText(str)
        .then(() => {
            ElMessage({
                type: 'success',
                message:'复制成功!'
            })
        })
        .catch(err => {
            // ... existing error handling ...
            ElMessage({
                type: 'error',
                message:'复制失败!'
            })
        })
}

const handleClear = () => {
    try{
        answers.value = Array(forCount.value).fill('')
        saveAnwser()
        ElMessage({
            type:'success',
            message:'清空成功!'
        })
    }
    catch{
        ElMessage({
            type:'error',
            message:'清空失败!'
        })
    }
}
</script>

<style scoped>

.operate-area {
    display: flex;
    width: 100%;
    justify-content: center;
    align-items: center;
    margin-top: 50px;
}
.copy{
    margin: 0 20px;
    margin-top: 20px;
}
.save {
    display: flex;
    width: 400px;
    height: fit-content;
    position: relative;
    top: 7px;
    
}
.loading {
    width: 50px;
    height: 50px;
}
.text-save {
    text-align: center;
}
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