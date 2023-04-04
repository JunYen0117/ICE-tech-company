<script setup>
// 進階加分版說明------------------------------------

// 與原版不一樣的地方：
// 1. 新增loopStarting, loopEnding, loadingItems 【33~36行程式碼】
// 2. 修改handleArray() 【64~86行程式碼】
// 3. 新增onMounted()，滾動滑鼠載入資料 【116~136行程式碼】

// 原先的程式碼的邏輯，會讀取data1全部資料（1000筆），分別和 data2, data3 進行比對，
// 組合成新陣到newArr，最後再渲染到畫面上。
// 進階版，則是透過常數loopStarting, loopEnding，我一次只取data1最前面100資料，
// 和 data2, data3 進行比對。
// 當畫面捲動到底部時，會改變loopStarting, loopEnding的值，於是再取出data1後面100筆料，進行比對，
// 以此類推……。利用一次只取100筆資料的方式，讓程式不要一次全部跑完data1的迴圈，來降低載入頁面時間。

// 此外，改寫handleArray()。原先是使用foreach方法，但在foreach裡面用if判斷式，
// 即使if條件已滿足了，迴圈仍不會中斷，會繼續跑下去。
// 現在進階版，改用for迴圈，可以在if判斷式中使用break，只要條件滿足，就中斷迴圈，減少迴圈執行的次數。

// 進階加分版說明Ending------------------------------------

import { ref, reactive, onBeforeMount, onMounted } from 'vue'

// 資料區域--------------------------------------
let data1 = []
let data2 = []
let data3 = {}

const newArr = reactive([])

const trSelected= ref('')

const loopStarting = ref(0)
const loopEnding = ref(99)

let loadingItems = false
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

	for(let i = loopStarting.value; i <= loopEnding.value; i++) {
		// data1與data2，利用"key"屬性互相比對
		for(let j = 0; j <= data2.length - 1; j++){
			if(data1[i].key === data2[j].key) {
				newArr.push({ ...data1[i], 'cell8': data2[j].cell8 })
				break
			}
		}

		// data1與data3，利用"cell4"屬性互相比對
		for(let k = 0; k <= data3Keys.length - 1; k++) {
			if(data1[i].cell4 === data3[data3Keys[k]].cell4) {
				newArr[i].cell9 = data3[data3Keys[k]].cell9
				break
			}
		}
	}
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

onMounted(() => {
	addEventListener('scroll', () => {
		const x = window.innerHeight
		const y = window.scrollY
		const z = document.querySelector('#table').clientHeight
		// 【x + y >= z】表示滑鼠快捲動到底部了
		if (x + y >= z && loadingItems !== true) {
			loadingItems = true
			// 如果迴圈索引值 還不等同於 整個資料長度
			// 就繼續新增資料到newArr，渲染到畫面上
			if (loopEnding.value !== data1.length -1) {
				loopStarting.value += 100
				loopEnding.value += 100
				handleArray()
			}
			setTimeout(() => {
          loadingItems = false
        }, 1000)
		}
	})
});

</script>

<template>
	<table id="table">
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
