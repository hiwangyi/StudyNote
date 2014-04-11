#Javascript中拷贝数组的方法

在平时的开发中，经常会碰到JS数组拷贝的问题。由于JS中的数组属于引用类型变量，因此不能直接通过赋值操作来进行拷贝，所以数组拷贝的问题实际上可以引申为对引用类型变量的拷贝问题。

我们已经知道，对于引用类型的变量拷贝需要使用深度拷贝，即对该变量的每一个成员都要深拷贝一份给一个新的同类型引用变量。基于此原则，可以得到一种最基础的数组拷贝方法，我们姑且称它为`直接复制法`。代码如下：
```js
/**
 * 直接复制法
 *
 * @return {Array} 原数组的拷贝
 */
Object.prototype.clone = function () {
    var me = this;
    var copy = {};
    if (Object.prototype.toString.call(me) === '[object Array]') {
        copy = [];
    }
    for (var i in me) {
        if (typeof me[i] === 'object') {
            copy[i] = me[i].clone();
        } else {
            copy[i] = me[i];
        }
    }
    return copy;
}
```
该方法清晰明了，不但可以实现数组拷贝，还可以实现对象的拷贝。测试代码如下：
```js
var array1 = [1, 2, 3, 4, 5];
var array2 = array1.clone();
console.log('array1:' + array1);
console.log('array2:' + array2);
array2.pop();
console.log('array1:' + array1);
console.log('array2:' + array2);
```
结果为：
```js
array1:1,2,3,4,5
array2:1,2,3,4,5
array1:1,2,3,4,5
array2:1,2,3,4
```
除了`直接复制法`，还有两种对数组进行拷贝的方法。一种是利用JSON.stringify和JSON.parse方法来实现，还有一种则是利用数组的join和字符串的split方法来实现。这两种方法本质都是先将要复制的数组转为字符串的形式，然后再把得到的字符串转化成新的数组。具体实现的代码如下：
```js
/**
 * JSON转化法
 *
 * @param {Array} arr 原数组
 * @return {Array} 原数组的拷贝
 */
function cloneArrayByJSON(arr) {
    if (!arr.length) {
        return [];
    }
    var arrayStr = JSON.stringify(arr);
    var copy = JSON.parse(arrayStr);
    return copy;
}

/**
 * 字符串转化法
 *
 * @param {Array} arr 原数组
 * @return {Array} 原数组的拷贝
 */
function cloneArrayByString(arr) {
    if (!arr.length) {
        return [];
    }
    var arrayStr = arr.join(',');   // 数组元素中不可以包含指定的分隔符‘,’
    var copy = arrayStr.split(','); // 否则split时得到的数组和原数组不同
    return copy;
}
```
这两种方法算是比较巧妙的利用了数组、字符串和JSON对象的特性，但需要注意的是，在`字符串转化法`中，要拷贝的数组**必须是字符串数组**，且选择的分隔符**不可以在数组元素中出现**，否则利用此法得到的结果是不正确的。

jQuery的extend方法也可以被用来实现数组的拷贝。`jQuery.extend([deep], target, object1[, objectN])`方法可以接受4个参数，`deep`表示是否进行深度拷贝（可选），`target`是拷贝结果对象，`object1`是要拷贝的对象，`objectN`表示可以将多个对象或者数组合并成一个（可选）。使用jQuery.extend方法拷贝数组的示例如下：
```js
var array1 = ['Zhao', 'Qian', 'Sun', 'Li'];
var array2 = [];
$.extend(true, array2, array1);
console.log('array1:' + array1);
console.log('array2:' + array2);
array2.push('Wang');
console.log('array1:' + array1);
console.log('array2:' + array2);
```
代码运行结果为：
```js
array1:Zhao,Qian,Sun,Li
array2:Zhao,Qian,Sun,Li
array1:Zhao,Qian,Sun,Li
array2:Zhao,Qian,Sun,Li,Wang
```
下面对文中所提到的4种拷贝数组的方法做一个性能测试。用来进行拷贝测试的数组有两个，一个是长度为10000的长字符串数组，另一个是长度为10000的对象数组，其中每个元素都是包含3层嵌套的对象。分别对这两个数组进行10000次与100次拷贝，测试代码和结果如下：
```js
/**
 * @file 数组拷贝方法性能测试
 * @environment OS X 10.9.2, 2.5 GHz Intel Core i5, Google Chrome 版本34.0.1847.116 
 * @date Fri Apr 11 2014
 */
var origin1 = [];
var origin2 = [];
var clone1, clone2;
var LENGTH = 10000;
var TIMES = 100;
var obj = {
    pro1: 1,
    pro2: '2',
    pro3: ['a', 'b', 'c'],
    pro4: {
        key1: 'wang',
        key2: [
            {
                name: 'John',
                age: 26,
                sex: 'male'
            },
            {
                name: 'Lucy',
                age: 24,
                sex: 'female'
            }
        ],
        key3: {
            one: 'a',
            two: 2,
            three: [3, 2, 1]
        }
    }
};
for (var i = 0; i < LENGTH; i++) {
    origin1.push('abcdefghijklmnopqrstuvwxyz1234567890');
    origin2.push(obj);
}
var start = +new Date();
for (i = 0; i < LENGTH; i++) {
    clone1 = origin1.clone(); // 直接复制法
}
var time1 = +new Date() - start;
for (i = 0; i < LENGTH; i++) {
    clone1 = cloneArrayByJSON(origin1); // JSON转化法
}
var time2 = +new Date() - start - time1;
for (i = 0; i < LENGTH; i++) {
    clone1 = cloneArrayByString(origin1); // 字符串转化法
}
var time3 = +new Date() - start - time1 - time2;
for (i = 0; i < LENGTH; i++) {
    clone1 = $.extend(true, [], origin1); // jQuery.extend拷贝法
}
var time4 = +new Date() - start - time1 - time2 - time3;
start = +new Date();
for (i = 0; i < TIMES; i++) {
    clone2 = origin2.clone(); // 直接复制法
}
var time5 = +new Date() - start;
for (i = 0; i < TIMES; i++) {
    clone2 = cloneArrayByJSON(origin2); // JSON转化法
}
var time6 = +new Date() - time5 - start;
for (i = 0; i < TIMES; i++) {
    clone2 = $.extend(true, {}, origin2); // jQuery.extend拷贝法
}
var time7 = +new Date() - time6 - time5 - start;
console.log('复制长字符串数组10000次\n====================');
console.log('1. 直接复制法：耗时 ' + time1 / 1000 + ' s');
console.log('2. JSON转化法：耗时 ' + time2 / 1000 + ' s');
console.log('3. 字符串转化法：耗时 ' + time3 / 1000 + ' s');
console.log('4. jQuery.extend拷贝法：耗时 ' + time4 / 1000 + ' s');
console.log('复制长对象数组100次\n=================');
console.log('5. 直接复制法：耗时 ' + time5 / 1000 + ' s');
console.log('6. JSON转化法：耗时 ' + time6 / 1000 + ' s');
console.log('7. jQuery.extend拷贝法：耗时 ' + time7 / 1000 + ' s');
```
测试结果：
```js
复制长字符串数组10000次
====================
1. 直接复制法：耗时 27.417 s
2. JSON转化法：耗时 34.661 s
3. 字符串转化法：耗时 14.685 s
4. jQuery.extend拷贝法：耗时 51.762 s
复制长对象数组100次
=================
5. 直接复制法：耗时 33.773 s
6. JSON转化法：耗时 7.168 s
7. jQuery.extend拷贝法：耗时 3.236 s
```
通过测试结果可以得出以下结论：

 1. 在针对长字符串数组的拷贝中，`字符串转化法`的拷贝速度最快，其次是`直接复制法`和`JSON转化法`，而使用`jQuery.extend`方法拷贝的速度是最慢的；在针对长对象数组的拷贝中，`jQuery.extend`方法拷贝的速度最快，`JSON转化法`次之，而`直接复制法`则最慢
 
 2. 因此当要大量复制长字符串数组时，使用`字符串转化法`能达到较好的时间性能；在复制由基本类型元素构成的数组时，选择`直接复制法`或`JSON转化法`比较合适；而在复制包含大量对象的数组时，`jQuery.extend`方法的时间性能最好，这一点将在数组元素较多、对象嵌套层次较深时体现的更为明显