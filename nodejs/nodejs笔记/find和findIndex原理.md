## 重写find,findIndex代码和find是一样的，只不过返回的是index
跟forEach差不多
```js
;['abc', 'd', 'efg'].forEach(function  (item, index) {
            console.log(item)
        })
```
```javascript
var users = [
    {id:1, name: '张三'},
    {id:2, name: '张三'},
    {id:3, name: '张三'},
    {id:4, name: '张三'}
]

// 函数作为参数
// 传条件函数
Array.prototype.myFind = function (conditionFunc) {
    // 相当于这里有一句这个代码：
    //conditionFunc = function (item, index) { return item.id===4}
    
    for (var i = 0; i < this.length; i++) {

        // 每循环一次要执行一次条件函数, 如果有一次执行返回的是ture就执行下面的代码
        if (conditionFunc(this[i], i)) {
            // 如果为true，就返回this[i]，遍历完都没有就是undefined
            return this[i]
        }
    }
}
var result = users.myFind(function (item, index) {
// 返回一个条件
    return item.id ===4
})

console.log(result)
```

## 重写forEach
```javascript
var users = [
    {id:1, name: '张三'},
    {id:2, name: '张三'},
    {id:3, name: '张三'},
    {id:4, name: '张三'}
]
Array.prototype.myEach = function(myFunc) {
    for (var i = 0; i < this.length; i++) {
        myFunc(this[i], i)
    }
}
users.myEach(function (item, index) {
    console.log(item)
})
```