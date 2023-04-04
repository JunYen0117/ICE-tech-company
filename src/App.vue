<script setup>
import { ref, reactive, onBeforeMount } from 'vue'

// 資料區域--------------------------------------
let data1 = []
let data2 = []
let data3 = {}

const newArr = reactive([])

const trSelected= ref('')
// 資料區域Ending----------------------------------

// 函式區域----------------------------------------------
// 第一步：先取得資料
const getAllData = async () => {
	try {
		// 取得data1.json
		const res1 = await fetch('../data/data1.json')
		if (!res1.ok) { throw new Error('get data1.json 出錯') }
		data1 = await res1.json()

		// 取得data2.json
		const res2 = await fetch('../data/data2.json')
		if (!res2.ok) { throw new Error('get data2.json 出錯') }
		data2 = await res2.json()

		// 取得data3.json
		const res3 = await fetch('../data/data3.json')
		if (!res3.ok) { throw new Error('get data3.json 出錯') }
		data3 = await res3.json()

	} catch (error) {
		console.log(error)
	}
}

// 第二步：操作陣列，結果存在newArr裡
const handleArray = () => {
	// data3Keys是data3的所有屬性，組成陣到 -> [91960, 1H1AO, YSOZD...]
	const data3Keys = Object.keys(data3)

	data1.forEach((data1Value, data1Index) => {
		// data1與data2，利用"key"屬性互相比對
		data2.forEach((data2Value, i) => {
			// data1Value.key -> R0, R1, R2...
			if (data1Value.key === data2Value.key) {
				// 【data1Value.key data2Value.cell8】 -> 【R0 GXYTD】,【R1 5FI13】...
				newArr.push({...data1Value, 'cell8': data2Value.cell8})
			}
		})

		// data1與data3，利用"cell4"屬性互相比對
		data3Keys.forEach((key, i) => {
			if (data1Value.cell4 === data3[key].cell4) {
				// newArr每個值（物件），裡面新增cell9屬性
				newArr[data1Index]['cell9'] = data3[key].cell9
			}
		})
	})
	console.log(newArr)
}

// 點擊<tr>觸發，單選
const changeTrSelected = (key) => {
	trSelected.value = key
}

// 點擊星星<span>觸發，多選
const changeSpanSelected = (key) => {
	const spanDom = document.querySelector(`#${key}`)
	const spanClassList = spanDom.classList
	// <span>裡的class屬性，有沒有'selected'值？已經有就移除；還沒有就新增
	if ( spanClassList.contains('selected') ) {
		spanClassList.remove('selected')
	} else {
		spanClassList.add('selected')
	}
}

// 先執行第一步getAllData，等資料都順利取得後，再執行第二步handleArray
const loadingSequence = async() => {
	await getAllData()
	await handleArray()
}
// 函式區域Ending----------------------------------------------

onBeforeMount(() => {
	loadingSequence()
});
</script>

<template>
	<table>
		<thead>
			<tr>
				<th>KEY</th>
				<th>CELL1</th>
				<th>CELL2</th>
				<th>CELL3</th>
				<th>CELL4</th>
				<th>CELL5</th>
				<th>CELL6</th>
				<th>CELL7</th>
				<th>CELL8</th>
				<th>CELL9</th>
			</tr>
		</thead>
		<tbody>
			<tr v-for="( item, index) in newArr" :key="item.key"
			  :class="{'selected': item.key === trSelected}"
				@click="changeTrSelected(item.key)">
				<td >
					<!-- @click.stop阻擋事件冒泡，避免點擊<span>，同時觸發<tr>的事件 -->
					<span class="star" :id="item.key"
					  @click.stop="changeSpanSelected(item.key)"></span>
					{{ item.key }}
				</td>
				<td>{{ item.cell1 }}</td>
				<td>{{ item.cell2 }}</td>
				<td>{{ item.cell3 }}</td>
				<td>{{ item.cell4 }}</td>
				<td>{{ item.cell5 }}</td>
				<td>{{ item.cell6 }}</td>
				<td>{{ item.cell7 }}</td>
				<td>{{ item.cell8 }}</td>
				<td>{{ item.cell9 }}</td>
			</tr>
		</tbody>
	</table>
</template>

<style scoped>
@import "./assets/main.css";
</style>
