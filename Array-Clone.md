#Javascript中拷贝数组的方法

在平时的开发中，经常会碰到JS数组拷贝的问题。由于JS中的数组属于引用类型变量，因此不能直接通过赋值操作来进行拷贝，所以数组拷贝的问题实际上可以引申为对引用类型变量的拷贝问题。
我们已经知道，对于引用类型的变量拷贝需要使用深度拷贝，即对该变量的每一个成员都要深拷贝一份给一个新的同类型引用变量。基于此原则，可以得到一种最基础的数组拷贝方法，我们姑且称它为**直接复制法**。代码如下：
`Object.prototype.clone = function () {
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
}`
该方法清晰明了，不但可以实现数组拷贝，还可以实现对象的拷贝。测试代码如下：
`var array1 = [1, 2, 3, 4, 5];
var array2 = array1.clone();
console.log('array1:' + array1);
console.log('array2:' + array2);
array2.pop();
console.log('array1:' + array1);
console.log('array2:' + array2);`
结果为：
`array1:1,2,3,4,5
array2:1,2,3,4,5
array1:1,2,3,4,5
array2:1,2,3,4`