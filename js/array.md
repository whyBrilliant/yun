# array

## array function



## array utils function

1. allEqual 检查数组各项是否相等

```javascript
const allEqual = arr => arr.every(val => val === arr[0])
```
2. approximatelyEqual 约等于

```javascript
const approximatelyEqual = (v1, v2, epsilon = 0.001) => Math.abs(v1 - v2) < epsilon;

approximatelyEqual(Math.PI / 2.0, 1.5708); // true
```



3. arrayToCSV 数组转CSV格式

```
const arrayToCSV = (arr, delimiter = ',') =>
  arr.map(v => v.map(x => `"${x}"`).join(delimiter)).join('\n');
  
arrayToCSV([['a', 'b'], ['c', 'd']]); // '"a","b"\n"c","d"'
arrayToCSV([['a', 'b'], ['c', 'd']], ';'); // '"a";"b"\n"c";"d"'
```



4. arrayToHtmlList: 数组转li列表
```
const arrayToHtmlList = (arr, listID) =>
  (el => (
    (el = document.querySelector('#' + listID)),
    (el.innerHTML += arr.map(item => `<li>${item}</li>`).join(''))
  ))();
  
arrayToHtmlList(['item 1', 'item 2'], 'myListID');
```



5. average：平均数

```
const average = (...nums) => nums.reduce((acc, val) => acc + val, 0) / nums.length;
average(...[1, 2, 3]); // 2
average(1, 2, 3); // 2
```



6. averageBy：数组对象属性平均数

```
const averageBy = (arr, fn) =>
  arr.map(typeof fn === 'function' ? fn : val => val[fn]).reduce((acc, val) => acc + val, 0) /
  arr.length;
  
averageBy([{ n: 4 }, { n: 2 }, { n: 8 }, { n: 6 }], o => o.n); // 5
averageBy([{ n: 4 }, { n: 2 }, { n: 8 }, { n: 6 }], 'n'); // 5
```



7. bifurcate 拆分断言后的数组

   可以根据每个元素的返回值，使用reduce和push将元素添加到第二次参数中

```
const bifurcate = (arr, filter) => 
	arr.reduce((acc, val, i) => (acc[filter[i] ? 0 : 1].push(val), acc), [[], []])
	
// [ ['beep', 'boop', 'bar'], ['foo'] ]
bifurcate(['beep', 'boop', 'foo', 'bar'], [true, true, false, trur])
```



8. countOccurrences 检测数值出现的次数

```
const countOccurences = (arr, val) => arr.reduce((a, v) => (v === val ? a + 1 : a), 0)
countOccurences([1, 1, 2, 1, 2, 3], 1)
```



9. deepFlatten 递归扁平化数组

```
// 方式一：
const deepFlatten = arr => [].concat(...arr.map(v => (Array.isArray(v) ? deepFlatten(v) : v)))

deepFlatten([1, [2], [[3], 4], 5]); // [1,2,3,4,5]

// 方式二：
a.toString().split(',').map(i => +i)	
```



10.  difference 

    查找两个数组之间的差异，并返回第一个数组独有的

```
const difference = (a, b) => {
	const s = new Set(b)
	return a.filter(x => !s.has(x))
}

difference([1, 2, 3], [1, 2, 4]) // [3]
```



11. differenceBy 先执行再寻找差异

```
const differenceBy = (a, b, fn) => {
	const s = new Set(b.map(fn))
	return a.filter(x => !s.has(fn(x)))
}

differenceBy([2.1, 1.2], [2.3, 2.4], Math.floor)
differenceBy([{ x: 2 }, { x: 1 }], [{ x: 1 }], v => v.x) // [{ x: 2 }]
```



12. dropWhile 删除不符合条件的值

```
const dropWhile = (arr, func) => {
  while (arr.length > 0 && !func(arr[0])) arr = arr.slice(1);
  return arr;
};

dropWhile([1, 2, 3, 4, 2], n => n >= 3); // [3,4,2]
```



13. depthFlatten

    指定深度的flatten

```
const depthFlatten = (arr, depth = 1) =>
	arr.reduce((a, v) => a.concat(depth > 1 && Array.isArray(v) ? flatten(v, depth - 1) : v), [])
	
depthFlatten([1, [2], 3, 4]) // [1, 2, 3, 4]
depthFlatten([1, [2, [3, [4, 5], 6], 7], 8], 2)// [1, 2, 3, [4, 5], 6, 7, 8]
```



14. indexOfAll 返回数组中某值的所有索引

```
const indexOfAll = (arr, val) => arr.reduce((acc, el, i) => (el === val ? [...acc, i] : acc), []);

indexOfAll([1, 2, 3, 1, 2, 3], 1); // [0,3]
indexOfAll([1, 2, 3], 4); // []
```



15. intersection 两数组的交集

```
const intersectionBy = (a, b, fn) => {
	const s = new Set(b.map(fn))
	return a.filter(x => s.has(fn(x)))
}

intersectionBy([2.1, 1.2], [2.3, 3.4], Math.floor) // [2.1]
```



16. intersectionWith 先比较后返回交集

```
const intersectionWith = (a, b, comp) => a.filter(x => b.findIndex(y => comp(x, y)) !== -1);

intersectionWith([1, 1.2, 1.5, 3, 0], [1.9, 3, 0, 3.9], (a, b) => Math.round(a) === Math.round(b)); // [1.5, 3, 0]
```



16. minN 返回指定长度的升序数组

```
const minN = (arr, n = 1) => [...arr].sort((a, b) => a - b).slice(0, n);

minN([1, 2, 3]); // [1]
minN([1, 2, 3], 2); // [1,2]
```



17. negate 根据条件反响筛选

```
const negate = func => (...args) => !func(...args);

[1, 2, 3, 4, 5, 6].filter(negate(n => n % 2 === 0)); // [ 1, 3, 5 ]
```



18. randomIntArrayInRange 生成两数之间指定长度的随机数组

```
const randomIntArrayInRange = (min, max, n = 1) =>
  Array.from({ length: n }, () => Math.floor(Math.random() * (max - min + 1)) + min);
  
randomIntArrayInRange(12, 35, 10); // [ 34, 14, 27, 17, 30, 27, 20, 26, 21, 14 ]
```



19. sample 在制定数组中获取随机数

```
const sample = arr => arr[Math.floor(Math.random() * arr.length)];

sample([3, 7, 9, 11]); // 9
```



20. sampleSize 在指定数组中获取指定长度的随机数

此代码段可用于从数组中获取指定长度的随机数，直至穷尽数组。 使用`Fisher-Yates`算法对数组中的元素进行随机选择。

```
const sampleSize = ([...arr], n = 1) => {
  let m = arr.length;
  while (m) {
    const i = Math.floor(Math.random() * m--);
    [arr[m], arr[i]] = [arr[i], arr[m]];
  }
  return arr.slice(0, n);
};

sampleSize([1, 2, 3], 2); // [3,1]
sampleSize([1, 2, 3], 4); // [2,3,1]
```



21. shuffle: "洗牌"数组

```

const shuffle = ([...arr]) => {
  let m = arr.length;
  while (m) {
    const i = Math.floor(Math.random() * m--);
    [arr[m], arr[i]] = [arr[i], arr[m]];
  }
  return arr;
};

const foo = [1, 2, 3];
shuffle(foo); // [2, 3, 1], foo = [1, 2, 3]
```



22. nest: 根据parent_id 生成树结构

```
const nest = (items, id = null, link = 'parent_id') =>
  items
    .filter(item => item[link] === id)
    .map(item => ({ ...item, children: nest(items, item.id) }));


// 示例
const comments = [
  { id: 1, parent_id: null },
  { id: 2, parent_id: 1 },
  { id: 3, parent_id: 1 },
  { id: 4, parent_id: 2 },
  { id: 5, parent_id: 4 }
];
const nestedComments = nest(comments); // [{ id: 1, parent_id: null, children: [...] }]
```

