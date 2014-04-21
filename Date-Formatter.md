#日期时间的格式化

在Web前端开发中，日期时间是处理和显示的最为频繁的一类信息。JS提供了原生的Date对象来处理与时间相关的操作，例如获取指定日期时间、年份月份、时间戳等功能，基本覆盖了对时间的所有操作。在获取了日期时间对象之后，还需要把它显示出来，`2014-04-21 14:07:23`和`2014年4月21日 14时07分23秒`等都是常见的日期显示格式。通常我们得到的都是统一的日期时间，如何将它们按照指定的不同格式显示出来，并且能够做到灵活可定制，则是一个值得解决的问题。

在研究了C#、Java等语言中的日期格式化函数之后，决定自己编写一个用于JS的日期时间格式化函数。编写的思路主要基于以下两点：
 1. 可对`Date`对象或日期时间数值直接进行格式化。能够适应不同的应用场景，方便使用；
 2. 格式化采用模板的方式。避免了复杂的字符串手动拼接操作，也便于设置输出日期时间的格式
根据上面的设计，给出一种时间格式化工具函数的实现，代码如下：
```js
var util = util || {};
util.Date = util.Date || {};
util.Date.formatter = function (template) {
    if (arguments.length <= 1) {
        console.error('error:Need more parameters!');
        return;
    }
    var datetime = [];
    var tplMap = {
        'y': 0,
        'M': 1,
        'd': 2,
        'H': 3,
        'm': 4,
        's': 5,
        'D': 6
    };
    var dayMap = ['日', '一', '二', '三', '四', '五', '六'];
    if (Object.prototype.toString.call(arguments[1]) === '[object Date]') {
        var obj = arguments[1];
        datetime[0] = obj.getFullYear();
        datetime[1] = obj.getMonth() + 1;
        datetime[2] = obj.getDate();
        datetime[3] = obj.getHours();
        datetime[4] = obj.getMinutes();
        datetime[5] = obj.getSeconds();
        datetime[6] = obj.getDay();
    } else {
        for (var i = 0, len = arguments.length - 1; i < len; i++) {
            datetime[i] = arguments[i + 1];
        }
    }
    var format = template.replace(/[A-z]%/g, function (match) {
        var mapIndex = tplMap[match.slice(0,1)];
        if (mapIndex === 6) {
            return dayMap[datetime[mapIndex]];
        }
        return datetime[mapIndex];
    });
    return format;
}
```
测试及结果：
```js
console.log(util.Date.formatter('y%年M%月d%日 星期D% H%时m%分s%秒', new Date()));
console.log(util.Date.formatter('y%-M%-d% H%:m%:s%', new Date()));
console.log(util.Date.formatter('y%/M%/d%', 2014, 11, 18));
console.log(util.Date.formatter('y%.M%.d% H%hm%ms%s', 2014, 3, 5, 6, 17, 28));

2014年4月21日 星期一 21时1分11秒
2014-4-21 21:1:11
2014/11/18
2014.3.5 6h17m28s
```
可以看到，`formatter`方法已经实现了基本的日期时间格式化功能，并且能直接new一个`Date`对象，输出为指定格式的时间。当然`formatter`方法并不完善，后续还可做一些改进工作，比如控制输出的月份、日期、小时为两位数，12小时制与24小时制的转换等。